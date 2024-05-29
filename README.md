# AutoReminderGoogleSheets
Google sheets project about auto organizing and reminding about Student Council Tech Event Requests

Link to project: https://docs.google.com/spreadsheets/d/1qb_PhxXqJlX4gIoOyaUYVAW7wZ8KNO9NJaUuK5XAGHM/edit?usp=sharing



Actual Code

// if you have any questions with the code email me
const sheet = SpreadsheetApp.getActiveSheet();

// this function is partly just to make code neater, just gets what is in the e column
function determineTask(index){
  return sheet.getRange("E" + index).getValue()
}

function checkWithinRange(date = new Date, limit){
  // takes the time from now to future date and converts into days, then checks if < limit and > 0
  return (date - new Date())/8.64e+7 <= limit && date - new Date() > 0
}

// at specific row return out the event in a stirng format, specifically for tech equipment
function formatEquipmentRequest(index){
  // gets Event Name, Location, and Date 
  const eventDetails = sheet.getRange("F" + index + ":H" + index).getValues()
  
  // checks if date is within time range, if so return the correct email format
  if (checkWithinRange(eventDetails[0][2], 30)){
    const timeUntil = (Math.round((eventDetails[0][2] - new Date())/8.64e+5)/100)
    var emailFormat = timeUntil + " Days until " + eventDetails[0][0] + ", tech equipment, at: " + eventDetails[0][1]
    return emailFormat
  } else{
    return ""
  }
}

// at specific row return out the event in a stirng format, specifically for Marquee
function formatMarqueeRequest(index = 46){
  // on a row get the advertisement name and starting date, returns a 2Darray of [[name, starting date]]
  const eventDetails = sheet.getRange("T" + index + ":U" + index).getValues()

  //checks the date in dex of the array and checks if it is within 30 days
  if (checkWithinRange(eventDetails[0][1], 30)){
    const timeUntil = (Math.round((eventDetails[0][1] - new Date())/8.64e+5)/100)
    var emailFormat = timeUntil + " Days until " + eventDetails[0][0] + " Advertisement"
    return emailFormat
  } else{
    return ""
  }
}

// at specific row return out the event in a stirng format, specifically for TV request
function formatTVRequest(index = 46){
  // gets slide and start date
  const eventDetails = sheet.getRange("Y" + index + ":Z" + index).getValues()

  // checks if within the date specified then returns the correct email format 
  if (checkWithinRange(eventDetails[0][1], 30)){
    const timeUntil = (Math.round((eventDetails[0][1] - new Date())/8.64e+5)/100)
    var emailFormat = timeUntil + " Days until TV request, slides are " + eventDetails[0][0]
    return emailFormat
  } else{
    return ""
  }
}

// links the proper request with the correct format function
function matchRequest(index){
  switch(determineTask(index)){
    case "Tech Equipment":
      return formatEquipmentRequest(index)

    case "Marquee Advertisement":
      return formatMarqueeRequest(index)

    case "TV Request":
      return formatTVRequest(index)

    default:
      return 
  }
}


// misleading name, gets all events then returns all in proper email format
function formatForEmails(){
  // creates list then adds the correct email format to the list all things
  var allThings = [];
  for (var y = 2; y <= sheet.getRange("AC2").getValue()+2; y++){
    if (matchRequest(y) != "" && matchRequest(y) != undefined)
      allThings.push(matchRequest(y))
  }

  // going to be real not sure what this line is doing, I think creating a sort function that sorts based on time
  console.log(allThings.sort((x,y) => x.split(" ")[0] - y.split(" ")[0]))
  
  // html tags are a bitch
  allThings = allThings.join("<br><br>")

  console.log(allThings)
  return allThings;
}

function checkDayOf(){
  // getting an array of all events
  const x = formatForEmails().split("<br><br>")
  var eventsToday = []
  // loops through all events and checks if the amount of days until is zero, if so add it to the list of events
  for (y = 0; y < x.length; y++){
    if (x[y][0] == 0 && x[y][1] < 7)
      eventsToday.push(x[y])
  }

  // checks if there is no events, if there isn't end function, if there is send email
  if (eventsToday.length === 0)
    return 0;

  else{
    MailApp.sendEmail({
      // change to current years tech and advisors **TODO**
      to: "kevinsandb3607@student.lvusd.org, kabirsingh1607@student.lvusd.org, mayankatte4707@student.lvusd.org, victorange9606@student.lvusd.org, cbuchanan@lvusd.org",
      subject: "Event(s) Today",
      htmlBody: eventsToday.join("<br><br>")
    })
  }
}

function sendEmails(){
  MailApp.sendEmail({
    
    // change to current years tech and advisors **TODO**
    to: "kevinsandb3607@student.lvusd.org, kabirsingh1607@student.lvusd.org, mayankatte4707@student.lvusd.org, victorange9606@student.lvusd.org",
    subject: "Upcoming Tech Events / Marquee Request / TV Requests",
    htmlBody: "<b> all upcoming jobs <b> <br><br>" + formatForEmails()
    });   
}

