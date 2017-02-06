# File Upload From Local HTML Form Using Google Web API

For the Web API of Google HTML service, cannot "multipart/form-data" be used from local HTML form? Although I investigated it and tried various methods of file upload using "multipart/form-data", all of them didn't work. So I thought other method which is file upload without "multipart/form-data". Of course, if you use Drive API, file upload can be easily done. But I wanted to achieve the file upload using Web API of Google HTML service. I think that this is self-satisfaction.

## Rule

1. Following GAS has to be made into a project of Google Apps Script.

2. Deploy the GAS project as a web application. [Ref](https://developers.google.com/apps-script/guides/web)

3. After updated the script, it has to be updated as a new version.

## GAS

This GAS is very simple sample. It uploads image files of bmp, gif, jpeg, png and svg. When you want to upload other files, please change following script.

```
function doGet(e) {
  return message("Error: no parameters");
}

function doPost(e) {
  if (!e.parameters.filename || !e.parameters.file || !e.parameters.imageformat) {
    return message("Error: Bad parameters");
  } else {
    var imgf = e.parameters.imageformat[0].toUpperCase();
    var mime =
        (imgf == 'BMP')  ? MimeType.BMP
      : (imgf == 'GIF')  ? MimeType.GIF
      : (imgf == 'JPEG') ? MimeType.JPEG
      : (imgf == 'PNG')  ? MimeType.PNG
      : (imgf == 'SVG')  ? MimeType.SVG
      : false;
    if (mime) {
      var data = Utilities.base64Decode(e.parameters.file, Utilities.Charset.UTF_8);
      var blob = Utilities.newBlob(data, mime, e.parameters.filename);
      DriveApp.getFolderById('FOLDER ID').createFile(blob);
      return message("completed");
    } else {
      return message("Error: Bad image format");
    }
  }
}

function message(msg) {
  return ContentService.createTextOutput(JSON.stringify({result: msg})).setMimeType(ContentService.MimeType.JSON);
}
```

Using "doPost" of [Google Web Apps](https://developers.google.com/apps-script/guides/web), files cannot be uploaded from local HTML form, while strings which isn't a file can be uploaded. So I thought that it converted from image data to base64 data, and upload it. By this, it was found that various files can be uploaded using "doPost" of Google Web Apps.

## html form on local PC

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <script src="http://code.jquery.com/jquery-latest.js" type="text/javascript"></script>
</head>
<body>
    <input type="file" id="file">

    <script type="text/javascript">
        $(function(){
            var url = 'https://script.google.com/macros/s/[Project ID]/exec';
            var params = {
                filename: 'samplefile',
                imageformat: 'PNG'
            };

            $('#file').on("change", function() {
                var file = this.files[0];
                var fr = new FileReader();
                fr.onload = function(e) {
                    params.file = e.target.result.replace(/^.*,/, '');
                    postJump();
                }
                fr.readAsDataURL(file);
            });

            function postJump(){
                var html = '<form method="post" action="'+url+'" id="postjump" style="display: none;">';
                Object.keys(params).forEach(function (key) {
                    html += '<input type="hidden" name="'+key+'" value="'+params[key]+'" >';
                });
                html += '</form>';
                $("body").append(html);
                $('#postjump').submit();
                $('#postjump').remove();
            }
        });
    </script>
</body>
</html>
```


