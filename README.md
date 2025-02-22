# msexcel-builder

A simple and fast library to create MS Office Excel(>2007) xlsx files(Compatible with the OpenOffice document format). 

Features:

* Support workbook and multi-worksheets.
* Custom column width and row height, cell range merge.
* Custom cell fill styles(such as background color).
* Custom cell border styles(such as thin,medium).
* Custom cell font styles(such as font-family,bold).
* Custom cell border styles and merge cells.
* Text rotation in cells.

## Getting Started

Install it in node.js:

```
npm install msexcel-builder
```

```javascript
var excelbuilder = require('msexcel-builder');
```


Then create a sample workbook with one sheet and some data.

```javascript
  // Create a new workbook file in current working-path
  var workbook = excelbuilder.createWorkbook('./', 'sample.xlsx')
  
  // Create a new worksheet with 10 columns and 12 rows
  var sheet1 = workbook.createSheet('sheet1', 10, 12);
  
  // Fill some data
  sheet1.set(1, 1, 'I am title');
  for (var i = 2; i < 5; i++)
    sheet1.set(i, 1, 'test'+i);
  
  // Save it
  workbook.save(function(err){
    if (err)
      throw err;
    else
      console.log('congratulations, your workbook created');
  });
```

or return a JSZip object that can be used to stream the contents (and even save it to disk):

```
workbook.generate(function(err, jszip) {
  if (err) throw err;
  else {
    jszip.generateAsync({type: "nodebuffer", compression: "DEFLATE"}).then(function(buffer) {
    require('fs').writeFile(workbook.fpath + '/' + workbook.fname, buffer, function (err) {});
  }
});
```

You can now provide the file path on save rather than in the constructor:

```js
   workbook.save("/tmp/workbook.xlsx", function(err) {
      if (err) throw err;
      console.log("open \"" + path + "\"");
   });
```

Further you can optionally compress the saved file:
```js
   workbook.save("/tmp/workbook.xlsx", {compressed: true}, function(err) {
      if (err) throw err;
      console.log("open \"" + path + "\"");
   });
```

## Use in browser

This depends on `xmlbuilder` and `jszip` which are included in the direct
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <script type="text/javascript" src="./xmlbuilder.js"></script>
  <script type="text/javascript" src="./jszip.js"></script>
  <script type="text/javascript" src="../lib/msexcel-builder.js"></script>
</head>
<body>
<script>
    var workbook = excelbuilder.createWorkbook()

    // Create a new worksheet with 10 columns and 12 rows
    var sheet1 = workbook.createSheet('sheet1', 10, 12);

    for (var i = 1; i < 10; i++) {
      for (var j = 1; j < 12; j++) {
        sheet1.set(i, j, i * j)
      }
    }

    workbook.generate(function (err, jszip) {
      if (err) return callback(err);

      jszip.generateAsync({type: "blob", mimeType: 'application/vnd.ms-excel;'}).then(function (blob) {
        var filename = 'test.xlsx'
        if (navigator.msSaveBlob) { // IE 10+
          navigator.msSaveBlob(blob, filename);
        } 
        else {
          var link = document.createElement("a");
          if (link.download !== undefined) { // feature detection
            // Browsers that support HTML5 download attribute
            var url = URL.createObjectURL(blob);
            link.setAttribute("href", url);
            link.setAttribute("download", filename);
            link.style.visibility = 'hidden';
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
          }
        }
      })
    });
  }
</script>
</body>
</html>
```

## API

### createWorkbook(save_path, file_name)

Create a new workbook file.

* `save_path` - (String) The path to save workbook.
* `file_name` - (String) The file name of workbook.

Returns a `Workbook` Object.

Example: create a xlsx file saved to `C:\test.xlsx`

```javascript
var workbook = excelbuilder.createWorkbook('C:\','test.xlsx');
```

### Workbook.createSheet(sheet_name,column_count,row_count)

Create a new worksheet with specified columns and rows

* `sheet_name` - (String) worksheet name.
* `column_count` - (Number) sheet column count.
* `row_count` - (Number) sheet row count.

Returns a `Sheet` object

Notes: The sheet name must be unique within a same workbook.  No error checking or collision-resolution mechanisms are currently applied.

Also the sheet name is cleaned, replacing disallowed characters `[]\/*:?` with a dash `-`

Example: Create a new sheet named 'sheet1' with 5 columns and 8 rows

```javascript
var sheet1 = workbook.createSheet('sheet1', 5, 8);
```

### Workbook.save(callback)

Save current workbook.

* `callback` - (Function) Callback function to handle save result.

Example:

```javascript
workbook.save(function(err){
  console.log('workbook saved ' + (err?'failed':'ok'));
});
```

### Workbook.cancel()

Cancel to make current workbook,drop all data.

### Sheet.set(col, row, val)

Set the cell data.

* `col` - (Number) Cell column index(start with 1).
* `row` - (Number) Cell row index(start with 1).
* `val` - (String) Cell data.  May be a string or number.

No returns.

Example:

```javascript
sheet1.set(1,1,'Hello ');
sheet1.set(2,1,'world!');
```

Date values are recognized.  If `val` is an instance of `Date` then 
the data is converted to an Excel value (e.g. `new Date('2016-06-23')` becomes `42544`)
and a date format is applied in Excel. 

__hack__
For some reason, the generated workbook only applies Date formats when the fill is also set.
So when a date value is set, the default format is filled with a white background.
You can override that with an explicit call to `fill`:
```javascript
    sheet1.set(1, 4, new Date('04/01/2009'))
    
    sheet1.set(1, 5, {
      set: new Date('04/01/2009'),
      fill: { type: "solid", fgColor: "FFAA000"},
      numberFormat:"m/d/yy"
    } )
```





### Sheet.set(col, row, obj)
You can also set objects as shorthand.  If the properties match a method
then the method will be called with that argument, e.g.

```js
 sheet1.set(1, 1, {
      set: 'Red, bold, italic, underlined and centered with border',
      font: {
        name: 'Verdana',
        sz: 32,
        color: "FF0022FF",
        bold: true,
        iter: true,
        underline: true
      },
      align: 'center',
      fill: {
        type: 'solid',
        fgColor: 'FFFF2200'
      }
    });


    sheet1.set(2, 2, {
      set: Math.PI,
      fill: {
        type: 'solid',
        fgColor: 'FF0022FF'
      },
      numberFormat: '0.00%'
    }) // 10=>'0.00%'


    sheet1.set(3, 3, {
      set: '' + Math.PI,
      fill: {
        type: 'solid',
        fgColor: '99BB66'
      }
    })
```    

### Sheet.formula(col, row, str) 
Create a cell formula
```
sheet1.formula(3, 3, "A3*B2")
```


### Sheet.width(col, width)
### Sheet.height(row, height)

Set the column width or row height

Example:

```javascript
sheet1.width(1, 30);
sheet1.height(1, 20);
```

### Sheet.align(col, row, align)
### Sheet.valign(col, row, valign)
### Sheet.wrap(col, row, wrap)
### Sheet.rotate(col, row, angle)

Set cell text align style and wrap style

* `align` - (String) align style: 'center'/'left'/'right'
* `valign` - (String) vertical align style: 'center'/'top'/'bottom'
* `wrap` - (String) text wrap style:'true' / 'false'
* `rotate` - (String) Numeric angle for text rotation: '90'/'-90'

Example:

```javascript
sheet1.align(2, 1, 'center');
sheet1.valign(3, 3, 'top');
sheet1.wrap(1, 1, 'true');
sheet1.rotate(1, 1, 90);
```

### Sheet.font(col, row, font_style)
### Sheet.fill(col, row, fill_style)
### Sheet.border(col, row, border_style)

Set cell font style, fill style or border style

* `font_style` - (Object) font style options 
The options may contain:

  * `name` - (String) font name
  * `sz` - (String) font size
  * `family` - (String) font family
  * `scheme` - (String) font scheme
  * `bold` - (String) if bold: 'true'/'false'
  * `iter || italic` - (String) if italic: 'true'/'false'
  * `underline`: (String) if underlined: 'true'/'false'
  * `strike`: (String) if striked: 'true'/'false'
  * `outline`: (String) if outline: 'true'/'false'
  * `shadow`: (String) if underlined: 'true'/'false'
  * `color` - (String) font color as HEX RGB or ARGB value (e.g. `"2266AA"` or `"FF2266AA`"`)

* `fill_style` - (Object) fill style options
The options may contain:

  * `type` - (String) fill type: such as 'solid'
  * `fgColor` - (String) front color, as HEX RGB or ARGB value (e.g. `"2266AA"` or `"FF2266AA`"`)
  * `bgColor` - (String) background color

* `border_style` - (Object) border style options
The options may contain:
  * `left` - (String) | (Object)
  * `top` - (String)  | (Object)
  * `right` - (String)  | (Object)
  * `bottom` - (String)  | (Obtject)
  
If (String) it may be `'thin'| 'medium'|'thick'|'double'`.

If (Object) it may be:
  `{
      style: <string>,
      color: { rgb: <string> } | { theme: <int> }`

 
Example:

```javascript
sheet1.font(2, 1, {name:'黑体',sz:'24',family:'3',scheme:'-',bold:'true',iter:'true', color: 'FFAA00'});
sheet1.fill(3, 3, {type:'solid',fgColor:'2266aa',bgColor:'64'});
sheet1.border(1, 1, {
  left:'medium',
  top: {
    style: 'medium',
    color: {rgb: "FFAA8844"}
  },
  right:'thin',
  bottom: {
    style: 'medium',
    color: {
      theme: 5
    }
  }}
)

```

#### Sheet.numberFormat(col, row, numfmt)
Specify a number format by string or index. Currently only Excel's built-in number formats are handled.

Example:
```javascript
sheet1.numberFormat(2,2, '0.00%');
sheet1.numberFormat(2,2, 10); // equivalent to above
```

```javascript
      0: 'General',
      1: '0',
      2: '0.00',
      3: '#,##0',
      4: '#,##0.00',
      9: '0%',
      10: '0.00%',
      11: '0.00E+00',
      12: '# ?/?',
      13: '# ??/??',
      14: 'm/d/yy',
      15: 'd-mmm-yy',
      16: 'd-mmm',
      17: 'mmm-yy',
      18: 'h:mm AM/PM',
      19: 'h:mm:ss AM/PM',
      20: 'h:mm',
      21: 'h:mm:ss',
      22: 'm/d/yy h:mm',
      37: '#,##0 ;(#,##0)',
      38: '#,##0 ;[Red](#,##0)',
      39: '#,##0.00;(#,##0.00)',
      40: '#,##0.00;[Red](#,##0.00)',
      45: 'mm:ss',
      46: '[h]:mm:ss',
      47: 'mmss.0',
      48: '##0.0E+0',
      49: '@',
      56: '"上午/下午 "hh"時"mm"分"ss"秒 "'
```


### Sheet.merge(from_cell, to_cell)

Merge some cell ranges

* `from_cell` / `to_cell` - (Object) cell position
The cell object contains:

  * `col` - (Number) cell column index(start with 1)
  * `row` - (Number) cell row index(start with 1) 

Example: Merge the first row as title from (1,1) to (5,1)

```javascript
sheet1.merge({col:1,row:1},{col:5,row:1});
```
### Sheet.autoFilter(filter_spec)
The argument may be a range (e.g. `"$A1:$J12"`) or `true` in which case the entire sheet domain is used as the range.

## Sheet.sheetViews(obj)

Optionally toggle grid lines and set zoom scale: 

    sheet1.sheetViews({
      showGridLines: "0",
      zoomScaleNormal: 50,
      zoomScale: 50
    })

### Sheet.split(ncols, nrows, state, activePane, topLeftCell)

Optionally freeze first rows and/or columns.  At a minimum specify the number of columns and rows.
The state defaults to "frozen", activePane to "bottomRight" and "topLeftCell" is calculated.

    sheet1.split(1, 2)
    sheet1.split(1, 2, "frozen", "bottomRight", "B2")

### Sheet.printBreakColumns([colIndexes])

Optionally set page breaks at specific columns
 
    sheet1.colBreaks([15,30,45])

### Sheet.printBreakRows([rowIndexes])

    sheet1.printBreakRows([15,30,45])

### Sheet.printRepeatRows(start, end)
Set rows to repeat on each printed page. 
Arguments may be specified individually or as an array of length 2.

     sheet.printRepeatRows(1,3)
     sheet.printRepeatRows([1,3])

### Sheet.printRepeatColumns(start, end)
Set columns to repeat on each printed page.
Arguments may be specified individually or as an array of length 2.
    sheet.printRepeatColumns(1,2)
    sheet.printRepeatColumns([1,2])

### Sheet.pageSetup(obj)

Optionally set paper size, orientation, and resolution:

    sheet1.pageSetup({
      paperSize: '9', 
      orientation: 'landscape' || 'portrait',
      horizontalDpi: '200', 
      verticalDpi: '200',
      pageOrder: 'overThenDown' ,
      scale: '50',
    })

### Sheet.pageMargins(obj)

Optionally set margins, units in inches:

    sheet.pageMargins({
      left: 0.25,
      right: 0.25,
      top: 0.5,
      bottom: 0.5,
      header: 0.3,
      footer: 0.3
    })

## Testing

There is a nascent Mocha test suite.
```
> npm test
```

A number of these tests currently writes output files and test for exact matches
against a reference file located in `test/files/`.

It's possible that a future feature extension might
break tests for innocent reasons (e.g. by writing additional XML to the workbook)
in which case, visually inspect the output file and update the reference file.

## Release notes

v0.3.10
* Add `Sheet.form(ncols, nrows)`

v0.3.9
* Add `Sheet.split(ncols, nrows)`

v0.3.8
* Add `Sheet.sheetViews` and `Sheet.pageSetup(obj)` 

v0.3.3
* Allow more than 26 columns
* Handle Javascript Date objects as cell values, converting to Excel dates
* Added concise form of `.set({})` which now also accepts an object

v0.3.0
*  Port to JSZip 3.1.2 (from 2.5), a breaking change which makes all JSZip methods asynchronous, 

v0.2.0
* Write numbers as numbers
* Write fill and font colors using hex ranges (ported from https://github.com/aloteot/msexcel-builder and applied into coffeescript)
* Apply autofilters
* Add mocha testing


v0.1.0
* Generate JSZip object, dropping need to generate temporary files on disk.
* Removed dependency on `fs-extra` and `exec` and `easy-zip`.
* Added dependency on `js-zip`.
* Removed method `save` and replaced it with `generate(callback)` that returns a JSZip object.
* This now theoretically should be able to run in the browser, though that is not tested.
* Also refactored base Excel files so they are read from code rather than from disk.

v0.0.2:
* Switch compress work to easy-zip to support Heroku deployment.

v0.0.1: Includes

* First release.
* Using 7z.exe to do compress work, so only support windows now.
