---
layout: post
title: PowerShell Universal Dashboard – Update Page On Button Click
---



I have recently gotten into working with PowerShell Universal Dashboard. At work we have some requirements that Universal Dashboard will help accommodate. So I got to work with the community edition to be able to do a proof of concept. Everything has gone very well and we are purchasing the suite. That being said the biggest issue I ran into was being able to update the page with results by pushing a button.

I knew that I wanted to create a new UDCard to display the information but I could not get it to display. That is when I figured out that I had to have a place holder UDElement and then once I had the UDCard I could just replace the place holder UDElement with the UDCard. To do this first I crated the new UDElement like this:

````
New-UDElement -Tag "PlaceHolder" -Id "results"
````

After that in the onClick for the button I replace the place holder with the card like this:

````
Set-UDElement -Id "results" -Content {
    New-UDCard -Title "Results" -Text "Your text"
}
````

In this example I have a Dashboard that has a text box for input of a name and then a button that when pushed will display a message in a new card.

````
#Get the existing running Dashboards and stop them.
Get-UDDashboard | Stop-UDDashboard

#Create the Dashboard
$Dashboard =  New-UDDashboard -Title "Test" -Content {
    #Create the text box
    New-UDTextbox -Id "txtName" -Label "Name" -Placeholder "Enter your name"
    #Create the button with onClick 
    New-UDButton -Id "btnSearch" -Text "Search" -OnClick {
        #Changes the UDElement "results" with the new UDCard
        Set-UDElement -Id "results" -Content {
            #Sets the varable Name with the value of the text box
            $Name = (Get-UDElement -Id "txtName").Attributes["value"]
            #Creates the UDCard with the title "Results" and the text "Your name is" followed by the text in the text box
            New-UDCard -Title "Results" -Text "Your name is $Name"
        }
    }
    #Place holder for the new UDCard
    New-UDElement -Tag "PlaceHolder" -Id "results"
}

#Starts the created Dashboard on port 80
Start-UDDashboard -Dashboard $Dashboard -Port 80
````

I hope this helps clarify the solution. I had a hard time finding anything about this since every example I have found just showed a UDToast and not actual output that staid on the screen.
