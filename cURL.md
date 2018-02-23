GET

    curl [URL]
    curl "http://www.hotmail.com/when/junk.cgi?birthyear=1905&press=OK"

POST

    curl --data "birthyear=1905&press=%20OK%20"  http://www.example.com/when.cgi

文件上传 POST

    curl --form upload=@localfilename --form press=OK [URL]

下载文件

    curl -O [URL]
