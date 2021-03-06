# Callback {#reference_zkm_311_hgb .reference}

To enable OSS to return callback information about an object to an application server after the object is uploaded to OSS, you just need to add a callback parameter in the upload request sent to OSS. This topic describes the implementation of upload callback in details.

## Background {#section_g4g_ydd_3gb .section}

-   APIs that support upload callback include: [PutObject](reseller.en-US/API Reference/Object operations/PutObject.md#), [PostObject](reseller.en-US/API Reference/Object operations/PostObject.md#), and [CompleteMultipartUpload](reseller.en-US/API Reference/Multipart upload operations/CompleteMultipartUpload.md#).
-   Regions that support upload callback include: China North 2 \(Beijing\), China East 1 \(Hangzhou\), China North 1 \(Qingdao\), China East 2 \(Shanghai\), China East 2 \(Finance Cloud\), China South 1 \(Finance Cloud\), China South 1 \(Shenzhen\), Hong Kong, China North 5 \(Hohhot\), China North 3 \(Zhangjiakou\), Middle East 1 \(Dubai\), Asia Pacific NE 1 \(Tokyo\), EU Central 1 \(Frankfurt\), Asia Pacific SE 1 \(Singapore\), US East 1 \(Virginia\), US West 1 \(Silicon Valley\), Asia Pacific SE 2 \(Sydney\), and Asia Pacific SE 3 \(Kuala Lumpur\).
-   For more information about upload callback, see [Principle](../../../../../reseller.en-US//Directly add a signature on the server, transfer the file, and set upload callback.md#).

## Step 1: Construct parameters. {#section_mnr_xbs_ggb .section}

-   Construct a callback parameter.

    A callback parameter is a base64-encoded string \(field\) in JSON format. To construct a callback parameter, it is important to specify the URL of the server to which the callback information is returned \(callbackUrl\) and the content of the callback information \(callbackBody\).

    The following table describes the JSON fields included in a callback parameter.

    |Field|Description|Required|
    |:----|:----------|:-------|
    |callbackUrl|     -   After an object is uploaded, OSS sends a callback request using the POST method to this URL. The body of the request is the content specified in callbackBody. This URL returns an `HTTP/1.1 200 OK` response only when the following conditions are met: 1. The body of the callback request is in JSON format. 2. The Content-Length header of the request must be a valid value smaller than 3 MB.
    -   You can set five URLs in a request in maximum by separating them by semicolons \(;\). OSS sends requests to each URL until the first success response is returned.
    -   If no URL is configured or the value of this field is null, OSS determines that the callback function is not configured.
    -   HTTPS IP addresses are supported.
    -   To ensure that Chinese characters can be correctly processed, the callbackUrl must be encoded. For example, if the value of callbackUrl is `http://example.com/Chinese characters.php?key=value&Chinese Name=Chinese Value`, it must be encoded into a JSON string, such as `http://example.com/%E4%B8%AD%E6%96%87.php?key=value&%E4%B8%AD%E6%96%87%E5%90%8D%E7%A7%B0=%E4%B8%AD%E6%96%87%E5%80%BC`.
 |Yes|
    |callbackHost|     -   Indicates the value of the Host header in the callback request. This field is valid only when the callbackUrl is specified.
    -   If this field is not specified, the hosts in the URLs specified in the callbackUrl field are resolved and specified as the value of this field.
 |No|
    |callbackBody|     -   Indicates the value of the callback request body, for example, as key=$\(key\)&etag=$\(etag\)&my\_var=$\(x:my\_var\).
    -   System variables, custom variables, and constants are supported for this field. Custom variables are passed through the callback-var parameter in PutObject and CompleteMultipart operations and through form fields in PostObject operations.
 |Yes|
    |callbackBodyType|     -   Indicates the Content-Type header in the callback request. This field supports two values: application/x-www-form-urlencoded and application/json, in which application/x-www-form-urlencoded is the default value.
    -   If the value of callbackBodyType is application/x-www-form-urlencoded, variables in callbackBody are replaced by the encoded URLs. If the value of callbackBodyType is application/json, the variables are replaced in JSON format.
 |No|

    Examples of the JSON fields included in a callback parameter are as follows:

    ```
    {
    "callbackUrl":"121.101.166.30/test.php",
    "callbackHost":"oss-cn-hangzhou.aliyuncs.com",
    "callbackBody":"{\"mimeType\":${mimeType},\"size\":${size}}",
    "callbackBodyType":"application/json"
    }
    ```

    ```
    {
    "callbackUrl":"121.43.113.8:23456/index.html",
    "callbackBody":"bucket=${bucket}&object=${object}&etag=${etag}&size=${size}&mimeType=${mimeType}&imageInfo.height=${imageInfo.height}&imageInfo.width=${imageInfo.width}&imageInfo.format=${imageInfo.format}&my_var=${x:my_var}"
    }
    ```

    The following table describes configurable system parameters in callbackBody.

    |System parameters|Description|
    |:----------------|:----------|
    |bucket|Indicates the bucket where the request object is stored.|
    |object|Indicates the requested object.|
    |etag|Indicates the ETag of the object, that is, the ETag field returned to the user who sends the request.|
    |size|Indicates the size of the requested object, which is the total object size in CompleteMultipartUpload operations.|
    |mimeType|Indicates the resource type. For example, the resource type of JPEG images is image/jpeg.|
    |imageInfo.height|Indicates the height of an image.|
    |imageInfo.width|Indicates the width of an image.|
    |imageInfo.format|Indicates the format of an image, such as jpg or png.|

    **Note:** Only an image object supports the imageinfo parameter. The values of imageInfo.height, imageInfo.width, imageInfo.format are null if the object is not an image.

-   Construct custom parameters using callback-var.

    You can configure custom parameters by using the callback-var parameter. A custom parameter is a key-value map. You can add required parameters to the map. When a POST callback request is initiated, OSS adds these custom parameters and the system parameters described in the preceding section to the body of the POST request, so that these parameters can be easily obtained by the user who sends the callback request.

    You can construct a custom parameter in the same way as you construct a callback parameter. A custom parameter is also a string in JSON format, which is a key-value map including all custom parameters.

    **Note:** The key of a custom parameter must start with "x:" and be lower-cased. Otherwise, OSS returns an error.

    Assume that you need to configure two custom parameters x:var1 and x:var2, and the values of the two parameters are value1 and value2 respectively. The constructed JSON strings are as follows:

    ```
    {
    "x:var1":"value1",
    "x:var2":"value2"
    }
    ```


**Note:** If the input callback parameter or callback-var parameter is invalid, a 400 error is returned with the InvalidArgument error code. This occurs in the following scenarios:

-   URLs and headers are input at the same time to the callback parameter \(x-oss-callback\) or the callback-var parameter \(x-oss-callback-var\) in PutObject and CompleteMultipartUpload operations.
-   The size of the callback or callback-var parameter \(this does not occur in PostObject operations because the callback-var parameter is not available in PostObject operations\) exceeds 5 KB.
-   The callback or callback-var parameter is not base64-encoded or is not in the valid JSON format after being decoded.
-   The callbackUrl field decoded from the callback parameter includes more than five URLs, or the port in the URL is invalid, for example:

    ```
    {"callbackUrl":"10.101.166.30:test",
        "callbackBody":"test"}
    ```

-   The callbackBody field decoded from the callback parameter is null.
-   The value of callbackBodyType decoded from the callback parameter is not `application/x-www-form-urlencoded` or `application/json`.
-   The variables in the callbackBody field decoded from the callback parameter are not in the valid format, that is, $\{var\}.
-   The callback-var parameter is not in the expected JSON format, that is, `{"x:var1":"value1","x:var2":"value2"...}`.

## Step 2: Construct a callback request. {#section_a45_yfx_wdb .section}

After constructing the callback and callback-var parameters, you must add the parameters to the callback request sent to OSS.

You can add the parameters in the following three methods:

-   Add the parameters to the URL.
-   Add the parameters to the header.
-   Add the parameters to the form fields in the body of a POST request.

    **Note:** You can use only this method to specify callback parameters when uploading objects using POST requests.


The preceding three methods are alternative. If you use more than one method, OSS returns an InvalidArgument error.

To add the parameters to a request sent to OSS, you must use base64 to encode the JSON string constructed in the preceding section, and then add the parameters as follows:

-   To add the parameters to the URL, add `callback=[CallBack]` or `callback-var=[CallBackVar]` to the request as a URL parameter. When the CanonicalizedResource field in the signature is calculated, callback or callback-var is used as a sub-resource.
-   To add the parameters to the header, add `x-oss-callback=[CallBack]` or `x-oss-callback-var=[CallBackVar]` to the request as a header. When the CanonicalizedOSSHeaders field in the signature is calculated, x-oss-callback-var and x-oss-callback are used. The code example is as follows:

    ```
    PUT /test.txt HTTP/1.1
    Host: callback-test.oss-test.aliyun-inc.com
    Accept-ncoding: identity
    Content-Length: 5
    x-oss-callback-var: eyJ4Om15X3ZhciI6ImZvci1jYWxsYmFjay10ZXN0In0=
    User-Agent: aliyun-sdk-python/0.4.0 (Linux/2.6.32-220.23.2.ali1089.el5.x86_64/x86_64;2.5.4)
    x-oss-callback: eyJjYWxsYmFja1VybCI6IjEyMS40My4xMTMuODoyMzQ1Ni9pbmRleC5odG1sIiwgICJjYWxsYmFja0JvZHkiOiJidWNrZXQ9JHtidWNrZXR9Jm9iamVjdD0ke29iamVjdH0mZXRhZz0ke2V0YWd9JnNpemU9JHtzaXplfSZtaW1lVHlwZT0ke21pbWVUeXBlfSZpbWFnZUluZm8uaGVpZ2h0PSR7aW1hZ2VJbmZvLmhlaWdodH0maW1hZ2VJbmZvLndpZHRoPSR7aW1hZ2VJbmZvLndpZHRofSZpbWFnZUluZm8uZm9ybWF0PSR7aW1hZ2VJbmZvLmZvcm1hdH0mbXlfdmFyPSR7eDpteV92YXJ9In0=
    Host: callback-test.oss-test.aliyun-inc.com
    Expect: 100-Continue
    Date: Mon, 14 Sep 2015 12:37:27 GMT
    Content-Type: text/plain
    Authorization: OSS mlepou3zr4u7b14:5a74vhd4UXpmyuudV14Kaen5cY4=
    Test
    ```

-   Use form fields to add parameters to the body of a POST request.
    -   It is slightly complicated to add the callback parameter when the POST method is used to upload an object because the callback parameter must be added using an independent form field, as shown in the following example:

        ```
        --9431149156168
        Content-Disposition: form-data; name="callback"
        eyJjYWxsYmFja1VybCI6IjEwLjEwMS4xNjYuMzA6ODA4My9jYWxsYmFjay5waHAiLCJjYWxsYmFja0hvc3QiOiIxMC4xMDEuMTY2LjMwIiwiY2FsbGJhY2tCb2R5IjoiZmlsZW5hbWU9JChmaWxlbmFtZSkmdGFibGU9JHt4OnRhYmxlfSIsImNhbGxiYWNrQm9keVR5cGUiOiJhcHBsaWNhdGlvbi94LXd3dy1mb3JtLXVybGVuY29kZWQifQ==
        ```

    -   Custom parameters cannot be added by including the callback-var parameter to a form field. Each custom parameter must be added by using an independent form field. For example, if the JSON string for the custom parameters is as follows:

        ```
        {
        "x:var1":"value1",
        "x:var2":"value2"
        }
        ```

        The form fields in the POST request are as follows:

        ```
        --9431149156168
        Content-Disposition: form-data; name="callback"
        eyJjYWxsYmFja1VybCI6IjEwLjEwMS4xNjYuMzA6ODA4My9jYWxsYmFjay5waHAiLCJjYWxsYmFja0hvc3QiOiIxMC4xMDEuMTY2LjMwIiwiY2FsbGJhY2tCb2R5IjoiZmlsZW5hbWU9JChmaWxlbmFtZSkmdGFibGU9JHt4OnRhYmxlfSIsImNhbGxiYWNrQm9keVR5cGUiOiJhcHBsaWNhdGlvbi94LXd3dy1mb3JtLXVybGVuY29kZWQifQ==
        --9431149156168
        Content-Disposition: form-data; name="x:var1"
        value1
        --9431149156168
        Content-Disposition: form-data; name="x:var2"
        value2
        ```

        You can also add callback conditions in the policy \(if callback parameters are not added, upload verification is not performed on this parameter\). For example:

        ```
        { "expiration": "2014-12-01T12:00:00.000Z",
          "conditions": [
            {"bucket": "johnsmith" },
            {"callback": "eyJjYWxsYmFja1VybCI6IjEwLjEwMS4xNjYuMzA6ODA4My9jYWxsYmFjay5waHAiLCJjYWxsYmFja0hvc3QiOiIxMC4xMDEuMTY2LjMwIiwiY2FsbGJhY2tCb2R5IjoiZmlsZW5hbWU9JChmaWxlbmFtZSkiLCJjYWxsYmFja0JvZHlUeXBlIjoiYXBwbGljYXRpb24veC13d3ctZm9ybS11cmxlbmNvZGVkIn0="},
            ["starts-with", "$key", "user/eric/"],
          ]
        }
        ```


## Step 3: Initiate a callback request. {#section_fgy_2hx_wdb .section}

If a file is uploaded successfully, OSS sends the content specified by the callback and callback-var parameters in the request to the application server by using the POST method as follows:

```
POST /index.html HTTP/1.0
Host: 121.43.113.8
Connection: close
Content-Length: 181
Content-Type: application/x-www-form-urlencoded
User-Agent: ehttp-client/0.0.1
bucket=callback-test&object=test.txt&etag=D8E8FCA2DC0F896FD7CB4CB0031BA249&size=5&mimeType=text%2Fplain&imageInfo.height=&imageInfo.width=&imageInfo.format=&x:var1=for-callback-test
```

## \(Optional\) Step 4: Sign the callback request. {#section_btz_phx_wdb .section}

If the callback parameter is configured in the request, OSS initiates a POST callback request to the application server through the URL specified by the callbackUrl field. To verify whether the callback request received by the application server is initiated by OSS, you can sign the callback request.

-   Generate a signature.

    A call request is signed by OSS using the RSA asymmetric algorithm.

    A signature is generated by encrypting the callback string with a private key, as shown in the following code:

    ```
    authorization = base64_encode(rsa_sign(private_key, url_decode(path) + query_string + ‘\n’ + body, md5))
    ```

    **Note:** In the preceding code, private\_key is a private key only known by OSS, path is the resource path included in the callback request, query\_string is the query string, and body is the message body specified in the callback request.

    A callback request is signed in the following steps:

    1.  Obtain the callback string to be signed, which is composed by the resource path obtained by decoding the URL, the original query string, a carriage return, and the callback message body.
    2.  Sign the callback string with the RSA encryption algorithm, that is, using the private key to encrypt the callback string. The hash function used for signature is MD5.
    3.  Use Base64 to encode the signed result to get the final signature and Add the signature to the authorization header in the callback request.
    The example of a signed request is as follows

    ```
    POST /index.php? id=1&index=2 HTTP/1.0
    Host: 121.43.113.8
    Connection: close
    Content-Length: 18
    authorization: kKQeGTRccDKyHB3H9vF+xYMSrmhMZjzzl2/kdD1ktNVgbWEfYTQG0G2SU/RaHBovRCE8OkQDjC3uG33esH2txA==
    Content-Type: application/x-www-form-urlencoded
    User-Agent: ehttp-client/0.0.1
    x-oss-pub-key-url: aHR0cDovL2dvc3NwdWJsaWMuYWxpY2RuLmNvbS9jYWxsYmFja19wdWJfa2V5X3YxLnBlbQ==
    bucket=yonghu-test
    ```

    In the preceding example, path is `/index.php`, query\_string is`? id=1&index=2`, and body is `bucket=yonghu-test`. The final signature is `kKQeGTRccDKyHB3H9vF+xYMSrmhMZjzzl2/kdD1ktNVgbWEfYTQG0G2SU/RaHBovRCE8OkQDjC3uG33esH2txA==`.

-   Verify the signature.

    Signature verification is an inverse process of signing a request. The signature is verified by the application server as follows:

    ```
    Result = rsa_verify(public_key, md5(url_decode(path) + query_string + ‘\n’ + body), base64_decode(authorization))
    ```

    The fields in the preceding code have the same meanings as they are used to sign the request, in which public\_key indicates the public key, authorization indicates the signature in the callback request header. The signature is verified as follows:

    1.  The x-oss-pub-key-url header in the callback request stores the base64-encoded URL of the public key. Therefore, you must decode the base64-coded URL to get the public key,

        ```
        public_key = urlopen(base64_decode(x-oss-pub-key-url header))
        ```

        **Note:** To ensure that the public key is issued by OSS, you must verify whether the value of the `x-oss-pub-key-url` header starts with `http://gosspublic.alicdn.com/` or `https://gosspublic.alicdn.com/`.

    2.  Obtain the decoded signature.

        ```
        signature = base64_decode(authorization header)
        ```

    3.  Obtain the string to be signed the same way as described in the process of signing the callback request.

        ```
        sign_str = url_decode(path) + query_string + ‘\n’ + body
        ```

    4.  Verify the signature.

        ```
        result = rsa_verify(public_key, md5(sign_str), signature)
        ```

    The preceding sample is used as an example:

    1.  Obtain the URL of the public key by base64-decoding `aHR0cDovL2dvc3NwdWJsaWMuYWxpY2RuLmNvbS9jYWxsYmFja19wdWJfa2V5X3YxLnBlbQ==`. The decoded URL is `http://gosspublic.alicdn.com/callback_pub_key_v1.pem`.
    2.  Base64-decode the signature header `kKQeGTRccDKyHB3H9vF+xYMSrmhMZjzzl2/kdD1ktNVgbWEfYTQG0G2SU/RaHBovRCE8OkQDjC3uG33esH2txA==`. \(The decoded result cannot be displayed because it is a nonprintable string.\)
    3.  Obtain the string to be signed, that is, url\_decode\(“index.php”\) + “?id=1&index=2” + “\\n” + “bucket=yonghu-test”, and perform the MD5 verification on the string.
    4.  Verify the signature.
-   Application server example

    The following Python code demonstrates how an application server verifies a signature. To run the code, the M2Crypto library must be installed.

    ```
    import httplib
    import base64
    import md5
    import urllib2
    from BaseHTTPServer import BaseHTTPRequestHandler, HTTPServer
    from M2Crypto import RSA
    from M2Crypto import BIO
    def get_local_ip():
        try:
            csock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            csock.connect(('8.8.8.8', 80))
            (addr, port) = csock.getsockname()
            csock.close()
            return addr
        except socket.error:
            return ""
    class MyHTTPRequestHandler(BaseHTTPRequestHandler):
        '''
        def log_message(self, format, *args):
            return
        '''
        def do_POST(self):
            #get public key
            pub_key_url = ''
            try:
                pub_key_url_base64 = self.headers['x-oss-pub-key-url']
                pub_key_url = pub_key_url_base64.decode('base64')
                if not pub_key_url.startswith("http://gosspublic.alicdn.com/") and not pub_key_url.startswith("https://gosspublic.alicdn.com/"):
                    self.send_response(400)
                    self.end_headers()
                    return
                url_reader = urllib2.urlopen(pub_key_url)
                #you can cache it
                pub_key = url_reader.read() 
            except:
                print 'pub_key_url : ' + pub_key_url
                print 'Get pub key failed!'
                self.send_response(400)
                self.end_headers()
                return
            #get authorization
            authorization_base64 = self.headers['authorization']
            authorization = authorization_base64.decode('base64')
            #get callback body
            content_length = self.headers['content-length']
            callback_body = self.rfile.read(int(content_length))
            #compose authorization string
            auth_str = ''
            pos = self.path.find('?')
            if -1 == pos:
                auth_str = urllib2.unquote(self.path) + '\n' + callback_body
            else:
                auth_str = urllib2.unquote(self.path[0:pos]) + self.path[pos:] + '\n' + callback_body
            print auth_str
            #verify authorization
            auth_md5 = md5.new(auth_str).digest()
            bio = BIO.MemoryBuffer(pub_key)
            rsa_pub = RSA.load_pub_key_bio(bio)
            try:
                result = rsa_pub.verify(auth_md5, authorization, 'md5')
            except:
                result = False
            if not result:
                print 'Authorization verify failed!'
                print 'Public key : %s' % (pub_key)
                print 'Auth string : %s' % (auth_str)
                self.send_response(400)
                self.end_headers()
                return
            #do something according to callback_body
            #response to OSS
            resp_body = '{"Status":"OK"}'
            self.send_response(200)
            self.send_header('Content-Type', 'application/json')
            self.send_header('Content-Length', str(len(resp_body)))
            self.end_headers()
            self.wfile.write(resp_body)
    class MyHTTPServer(HTTPServer):
        def __init__(self, host, port):
            HTTPServer.__init__(self, (host, port), MyHTTPRequestHandler)
    if '__main__' == __name__:
        server_ip = get_local_ip()
    server_port = 23451
    server = MyHTTPServer(server_ip, server_port)
    server.serve_forever()
    ```

    The code for the application server in other languages is as follows

    Java:

    -   Click [here](https://gosspublic.alicdn.com/images/AppCallbackServer.zip) to download the code.
    -   Running method: Extract the package and run `java -jar oss-callback-server-demo.jar 9000` \(9000 is the port number and can be specified as needed\).
    PHP:

    -   Click [here](https://gosspublic.alicdn.com/callback-php-demo.zip) to download the code.
    -   Running method: Deploy the code to an Apache environment because some headers in the PHP code is depended on the environment. You can modify the example code according to the environment.
    Python:

    -   Click [here](https://gosspublic.alicdn.com/images/callback_app_server.py.zip) to download the code.
    -   Running method: Extract the package and run `python callback_app_server.py`. To run the code, RSA dependencies must be installed.
    C\#:

    -   Click [here](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/31989/cn_zh/1501048926621/callback-server-dotnet.zip) to download the code.
    -   Running method: Extract the package and see `README.md` to get more information.
    .NET :

    -   Click [here](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/31989/cn_zh/1501048926621/callback-server-dotnet.zip) to download the code.
    -   Running method: Extract the package and see `README.md` to get more information.
    Go:

    -   Click [here](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/31989/cn_zh/1501048745465/callback-server-go.zip) to download the code.
    -   Running method: Extract the package and see `README.md` to get more information.
    Ruby:

    -   Click [here](https://github.com/rockuw/oss-callback-server) to download the code.
    -   Running method: Run ruby aliyun\_oss\_callback\_server.rb.

## Step 5: Return the callback result. {#section_ald_hhx_wdb .section}

The application server returns the response to OSS.

The response to the callback request is as follows:

```
HTTP/1.0 200 OK
Server: BaseHTTP/0.3 Python/2.7.6
Date: Mon, 14 Sep 2015 12:37:27 GMT
Content-Type: application/json
Content-Length: 9
{"a":"b"}
```

**Note:** The response returned by the application server to OSS must contain the Content-Length header, and the size of the response body cannot exceed 1 MB.

## Step 6: Return the upload result. {#section_rhz_jhx_wdb .section}

OSS returns the information returned by the application server to the user.

The returned response is as follows:

```
HTTP/1.1 200 OK
Date: Mon, 14 Sep 2015 12:37:27 GMT
Content-Type: application/json
Content-Length: 9
Connection: keep-alive
ETag: "D8E8FCA2DC0F896FD7CB4CB0031BA249"
Server: AliyunOSS
x-oss-bucket-version: 1442231779
x-oss-request-id: 55F6BF87207FB30F2640C548
{"a":"b"}
```

**Note:** 

-   The body of responses for some requests \(such as CompleteMultipartUpload\) contains content \(for example, information in XMl format\). If you use the upload callback function, the original body content is overwritten, such as `{"a":"b"}`. Take this into consideration when you use the upload callback function.
-   If the upload callback fails, a 203 error is returned with the error code CallbackFailed. This indicates that the file is successfully uploaded to OSS but the callback fails. A callback failure only indicates that OSS does not receive the expected callback response, but not indicates that the application server does not receive a callback request. For example, the response returned by the application server is not in JSON format.

