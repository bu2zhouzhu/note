# 层次状态机(HMS)

frameworks/base/core/java/com/android/internal/util/StateMachine.java

The state machine defined here is a hierarchical state machine which processes messages and can have states arranged hierarchically.

A state is a `State` object and must implement `processMessage` and optionally `enter/exit/getName`. The enter/exit methods are equivalent to the construction and destruction in Object Oriented programming and are used to perform initialization and cleanup of the state respectively. The `getName` method returns the name of the state; the default implementation returns the class name. It may be desirable to have `getName` return the the state instance name instead, in particular if a particular state class has multiple instances.

When a state machine is created, `addState` is used to build the hierarchy and `setInitialState` is used to identify which of these is the initial state. After construction the programmer calls `start` which initializes and starts the state machine. The first action the StateMachine is to the invoke `enter` for all of the initial state's hierarchy, starting at its eldest parent. The calls to enter will be done in the context of the StateMachine's Handler, not in the context of the call to start, and they will be invoked before any messages are processed. For example, given the simple state machine below, mP1.enter will be invoked and then mS1.enter. Finally, messages sent to the state machine will be processed by the current state; in our simple state machine below that would initially be mS1.processMessage.

```
        mP1
       /   \
      mS2   mS1 ----> initial state

```

After the state machine is created and started, messages are sent to a state machine using `sendMessage` and the messages are created using `obtainMessage`. When the state machine receives a message the current state's `processMessage` is invoked. In the above example mS1.processMessage will be invoked first. The state may use `transitionTo` to change the current state to a new state.

Each state in the state machine may have a zero or one parent states. If a child state is unable to handle a message it may have the message processed by its parent by returning false or NOT_HANDLED. If a message is not handled by a child state or any of its ancestors, `unhandledMessage` will be invoked to give one last chance for the state machine to process the message.

When all processing is completed a state machine may choose to call `transitionToHaltingState`. When the current `processingMessage` returns the state machine will transfer to an internal `HaltingState` and invoke `halting`. Any message subsequently received by the state machine will cause `haltedProcessMessage` to be invoked.

If it is desirable to completely stop the state machine call `quit` or `quitNow`. These will call `exit` of the current state and its parents, call `onQuitting` and then exit Thread/Loopers.

In addition to `processMessage` each `State` has an `enter` method and `exit` method which may be overridden.

Since the states are arranged in a hierarchy transitioning to a new state causes current states to be exited and new states to be entered. To determine the list of states to be entered/exited the common parent closest to the current state is found. We then exit from the current state and its parent's up to but not including the common parent state and then enter all of the new states below the common parent down to the destination state. If there is no common parent all states are exited and then the new states are entered.

Two other methods that states can use are `deferMessage` and `sendMessageAtFrontOfQueue`. The `sendMessageAtFrontOfQueue` sends a message but places it on the front of the queue rather than the back. The`deferMessage` causes the message to be saved on a list until a transition is made to a new state. At which time all of the deferred messages will be put on the front of the state machine queue with the oldest message at the front. These will then be processed by the new current state before any other messages that are on the queue or might be added later. Both of these are protected and may only be invoked from within a state machine.

To illustrate some of these properties we'll use state machine with an 8 state hierarchy:

```
          mP0
         /   \
        mP1   mS0
       /   \
      mS2   mS1
     /  \    \
    mS3  mS4  mS5  ---> initial state

```

After starting mS5 the list of active states is mP0, mP1, mS1 and mS5. So the order of calling processMessage when a message is received is mS5, mS1, mP1, mP0 assuming each processMessage indicates it can't handle this message by returning false or NOT_HANDLED.

Now assume mS5.processMessage receives a message it can handle, and during the handling determines the machine should change states. It could call transitionTo(mS4) and return true or HANDLED. Immediately after returning from processMessage the state machine runtime will find the common parent, which is mP1. It will then call mS5.exit, mS1.exit, mS2.enter and then mS4.enter. The new list of active states is mP0, mP1, mS2 and mS4. So when the next message is received mS4.processMessage will be invoked.

Now for some concrete examples, here is the canonical HelloWorld as a state machine. It responds with "Hello World" being printed to the log for every message.

```java
class HelloWorld extends StateMachine {
    HelloWorld(String name) {
        super(name);
        addState(mState1);
        setInitialState(mState1);
    }
    public static HelloWorld makeHelloWorld() {
        HelloWorld hw = new HelloWorld("hw");
        hw.start();
        return hw;
    }
    class State1 extends State {
        @Override public boolean processMessage(Message message) {
            log("Hello World");
            return HANDLED;
        }
    }
    State1 mState1 = new State1();
}
void testHelloWorld() {
    HelloWorld hw = makeHelloWorld();
    hw.sendMessage(hw.obtainMessage());
}

```

A more interesting state machine is one with four states with two independent parent states.

```
        mP1      mP2
       /   \
      mS2   mS1

```

Here is a description of this state machine using pseudo code.

```
state mP1 {
     enter { log("mP1.enter"); }
     exit { log("mP1.exit");  }
     on msg {
         CMD_2 {
             send(CMD_3);
             defer(msg);
             transitionTo(mS2);
             return HANDLED;
         }
         return NOT_HANDLED;
     }
}
INITIAL
state mS1 parent mP1 {
     enter { log("mS1.enter"); }
     exit  { log("mS1.exit");  }
     on msg {
         CMD_1 {
             transitionTo(mS1);
             return HANDLED;
         }
         return NOT_HANDLED;
     }
}
state mS2 parent mP1 {
     enter { log("mS2.enter"); }
     exit  { log("mS2.exit");  }
     on msg {
         CMD_2 {
             send(CMD_4);
             return HANDLED;
         }
         CMD_3 {
             defer(msg);
             transitionTo(mP2);
             return HANDLED;
         }
         return NOT_HANDLED;
     }
}
state mP2 {
     enter {
         log("mP2.enter");
         send(CMD_5);
     }
     exit { log("mP2.exit"); }
     on msg {
         CMD_3, CMD_4 { return HANDLED; }
         CMD_5 {
             transitionTo(HaltingState);
             return HANDLED;
         }
         return NOT_HANDLED;
     }
}

```

The implementation is below and also in StateMachineTest:

```java
class Hsm1 extends StateMachine {
    public static final int CMD_1 = 1;
    public static final int CMD_2 = 2;
    public static final int CMD_3 = 3;
    public static final int CMD_4 = 4;
    public static final int CMD_5 = 5;
    public static Hsm1 makeHsm1() {
        log("makeHsm1 E");
        Hsm1 sm = new Hsm1("hsm1");
        sm.start();
        log("makeHsm1 X");
        return sm;
    }
    Hsm1(String name) {
        super(name);
        log("ctor E");
        // Add states, use indentation to show hierarchy
        addState(mP1);
            addState(mS1, mP1);
            addState(mS2, mP1);
        addState(mP2);
        // Set the initial state
        setInitialState(mS1);
        log("ctor X");
    }
    class P1 extends State {
        @Override public void enter() {
            log("mP1.enter");
        }
        @Override public boolean processMessage(Message message) {
            boolean retVal;
            log("mP1.processMessage what=" + message.what);
            switch(message.what) {
            case CMD_2:
                // CMD_2 will arrive in mS2 before CMD_3
                sendMessage(obtainMessage(CMD_3));
                deferMessage(message);
                transitionTo(mS2);
                retVal = HANDLED;
                break;
            default:
                // Any message we don't understand in this state invokes unhandledMessage
                retVal = NOT_HANDLED;
                break;
            }
            return retVal;
        }
        @Override public void exit() {
            log("mP1.exit");
        }
    }
    class S1 extends State {
        @Override public void enter() {
            log("mS1.enter");
        }
        @Override public boolean processMessage(Message message) {
            log("S1.processMessage what=" + message.what);
            if (message.what == CMD_1) {
                // Transition to ourself to show that enter/exit is called
                transitionTo(mS1);
                return HANDLED;
            } else {
                // Let parent process all other messages
                return NOT_HANDLED;
            }
        }
        @Override public void exit() {
            log("mS1.exit");
        }
    }
    class S2 extends State {
        @Override public void enter() {
            log("mS2.enter");
        }
        @Override public boolean processMessage(Message message) {
            boolean retVal;
            log("mS2.processMessage what=" + message.what);
            switch(message.what) {
            case(CMD_2):
                sendMessage(obtainMessage(CMD_4));
                retVal = HANDLED;
                break;
            case(CMD_3):
                deferMessage(message);
                transitionTo(mP2);
                retVal = HANDLED;
                break;
            default:
                retVal = NOT_HANDLED;
                break;
            }
            return retVal;
        }
        @Override public void exit() {
            log("mS2.exit");
        }
    }
    class P2 extends State {
        @Override public void enter() {
            log("mP2.enter");
            sendMessage(obtainMessage(CMD_5));
        }
        @Override public boolean processMessage(Message message) {
            log("P2.processMessage what=" + message.what);
            switch(message.what) {
            case(CMD_3):
                break;
            case(CMD_4):
                break;
            case(CMD_5):
                transitionToHaltingState();
                break;
            }
            return HANDLED;
        }
        @Override public void exit() {
            log("mP2.exit");
        }
    }
    @Override
    void onHalting() {
        log("halting");
        synchronized (this) {
            this.notifyAll();
        }
    }
    P1 mP1 = new P1();
    S1 mS1 = new S1();
    S2 mS2 = new S2();
    P2 mP2 = new P2();
}

```

If this is executed by sending two messages CMD_1 and CMD_2 (Note the synchronize is only needed because we use hsm.wait())

```java
Hsm1 hsm = makeHsm1();
synchronize(hsm) {
     hsm.sendMessage(obtainMessage(hsm.CMD_1));
     hsm.sendMessage(obtainMessage(hsm.CMD_2));
     try {
          // wait for the messages to be handled
          hsm.wait();
     } catch (InterruptedException e) {
          loge("exception while waiting " + e.getMessage());
     }
}

```

The output is:

```
D/hsm1    ( 1999): makeHsm1 E
D/hsm1    ( 1999): ctor E
D/hsm1    ( 1999): ctor X
D/hsm1    ( 1999): mP1.enter
D/hsm1    ( 1999): mS1.enter
D/hsm1    ( 1999): makeHsm1 X
D/hsm1    ( 1999): mS1.processMessage what=1
D/hsm1    ( 1999): mS1.exit
D/hsm1    ( 1999): mS1.enter
D/hsm1    ( 1999): mS1.processMessage what=2
D/hsm1    ( 1999): mP1.processMessage what=2
D/hsm1    ( 1999): mS1.exit
D/hsm1    ( 1999): mS2.enter
D/hsm1    ( 1999): mS2.processMessage what=2
D/hsm1    ( 1999): mS2.processMessage what=3
D/hsm1    ( 1999): mS2.exit
D/hsm1    ( 1999): mP1.exit
D/hsm1    ( 1999): mP2.enter
D/hsm1    ( 1999): mP2.processMessage what=3
D/hsm1    ( 1999): mP2.processMessage what=4
D/hsm1    ( 1999): mP2.processMessage what=5
D/hsm1    ( 1999): mP2.exit
D/hsm1    ( 1999): halting
```