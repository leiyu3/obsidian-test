---
title: How To Create Google Calendar Events in Bulk
cdate: 2022-06-19
mdate: 2022-06-19
categories:
  - how-to
publish: true
---

I am responsible for updating my church's fellowship schedules on the church website. We were using google sheet to share the schedules. But I recently added embedded google calendars to the page to replace the google sheets, I believe that it is a much more elegant solution. But with this change I need a faster way to create events to google calendar. And I have found a way to allow me to create google calendars in bulk. The steps are written [below.](How%2520To%2520Create%2520Google%2520Calendar%2520Events%2520in%2520Bulk.md##method) You can see the web page showing the fellowship schedules [here.](https://chinesegospelchurch.net/cgcse/ministries/fellowships/)

## Method

Add your calendar event in this [Google Sheet Template](https://docs.google.com/spreadsheets/d/1gz1PE-WwtfIlhR_bKiWW93Vq_DmK1dtgCnKzZPR3nIk/edit?usp=sharing). You probably need to make your own copy to be able to edit it. To do that, click `File` &rarr; `Make a Copy`.

After adding your calendar events, export as `.csv` file (`File` &rarr; `Download` &rarr; `Comma Separated Values`).

Go to Google Calendar. Click on the gear icon on the top right. Go to `Settings`. Then on the right click on `Import & export`. Or simply use this [link](https://calendar.google.com/calendar/u/0/r/settings/export) if you are logged in to the right account.

Then, under `Import` select the `CSV` file that you exported. And select the calendar that you want to bulk add the event to. Finally, press the `Import` button.

And your calendar events should be created on to the calendar that you selected.

If there is any errors that came up when you're importing the `CSV` file. Check out the [doc](https://support.google.com/calendar/answer/37118?hl=en&co=GENIE.Platform%3DDesktop). Make sure that your headings are set correctly and the required fields are set.


### Format of the CSV File
| Subject | Start Date | Start Time |
| ---- | ---- | ---- |
| `text` | `mm/dd/yyyy` | `hh:mm:ss AM/PM` |
