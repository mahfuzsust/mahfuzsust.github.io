---
title: Log everyday in google spreadsheet
slug: log-everyday-in-google-spreadsheet
date: 2021-03-24T04:16:13.000Z
date_updated: 2021-04-20T06:09:48.000Z
tags: 
  - app script
  - spreadsheet
---

I just wanted to create a daily task list with notes and reminders in google spreadsheet. So, I created a template which will look like this.
![](/content/images/2021/03/Screen-Shot-2021-03-24-at-9.44.06-AM.png)
Now, I need to create a new sheet everyday where the sheet name will denote the day and after each month I needed it to back up in my google drive.

To solve this problem, I have created a app script which will create a new sheet everyday I open the spreadsheet and backup in a separate file.

My tasks will be

- Create a new clone of the template everyday
- Backup current sheet monthly
- Create google drive folders to organize

First, we need to create a clone of the template sheet.
{% codeblock lang:java %}
function cloneTemplateDaily() {
  var name = Utilities.formatDate(new Date(), "GMT", "dd-MM-yyyy");
  
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var duplicate_sheet = ss.getSheetByName(name); 
  if(duplicate_sheet == null) { 
    var templateSheet = ss.getSheetByName("template");
    ss.setActiveSheet(templateSheet);
    duplicate_sheet = ss.duplicateActiveSheet();
    duplicate_sheet.setName(name);
  }
  ss.setActiveSheet(duplicate_sheet);
}
{% endcodeblock %}


Then I need to backup the sheet to my google drive
{% codeblock lang:java %}
function backupSheetMonthly() {
  var lastMonthDate = getLastMonthDate();
  var fileName = Utilities.formatDate(lastMonthDate, "GMT", "MMMM, yyyy");
  var year = Utilities.formatDate(lastMonthDate, "GMT", "yyyy");

  var rootFolder = getFolder("My Logs");
  var yearFolder = getFolder(year, rootFolder);

  if(!fileExist(fileName, yearFolder)) {
    var file = DriveApp.getFileById(SpreadsheetApp.getActiveSpreadsheet().getId())
    file.makeCopy(fileName, yearFolder);
    deleteSheets();
  }
}
{% endcodeblock %}


Now, all the methods that I require
{% codeblock lang:java %}
function fileExist(name, folder) {
  var files = folder.getFilesByName(name);
  while(files.hasNext()){
    var file = files.next();
    if(file.getName() == month) {
      return true;
    }
  };
  return false;
}

function getFolder(name, parent) {
  var folders = null;
  if(!parent) {
    folders = DriveApp.getFoldersByName(name);
  }
  else {
    folders = parent.getFoldersByName(name);
  }
  
  while(folders.hasNext()){
    var folder = folders.next();
    if(folder.getName() == name) {
      return folder;
    }
  }

  if(!parent) {
    return DriveApp.createFolder(name);
  }
  return parent.createFolder(name);
}

function getLastMonthDate() {
  var dt = new Date();
  dt.setDate(0);
  dt.setDate(1);
  return dt;
}

function deleteSheets() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheets = ss.getSheets();
  for(var i = 0; i < sheets.length; i++) {
    if(sheets[i].getName() !== "template") {
      ss.setActiveSheet(sheets[i]);
      ss.deleteActiveSheet();
      Utilities.sleep(100);
    }
  }
}
{% endcodeblock %}


Lastly, I created two separate trigger that will call my two functions

1. cloneTemplateDaily
2. backupSheetMonthly

I set up a trigger poninted to From spreadsheet - On open cloneTemplateDaily so there will be a new sheet name current date with the same template

And a time based trigger that will run 1st of every month to run backupSheetMonthly function.
