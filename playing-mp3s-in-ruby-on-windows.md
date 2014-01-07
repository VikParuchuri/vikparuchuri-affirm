Title: Playing MP3s in Ruby on Windows
Date: 2012-01-10 18:38
Slug: playing-mp3s-in-ruby-on-windows
Modified: 2014-01-07 15:30
Status: published
Category: 
Tags: Ruby, Windows


<div class='post'>
I recently needed to play music in Ruby in order to create an alarm clock of sorts. The code to do this was (surprise surprise) fairly simple, but I want to post it for posterity. <pre class="ruby"><br />require 'win32ole'<br />player = WIN32OLE.new('WMPlayer.OCX')<br />player.OpenPlayer('C:\alarm.mp3')<br /></pre> This will open a new instance of Windows Media Player that plays the selected song.<br><br> Note that win32ole should be installed by default, and does not need to be installed as a gem.</div>