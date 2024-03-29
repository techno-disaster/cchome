# Subtitle Downloader Technical Documentation

This page contains how the service modules were coded and also how to
add support for a new service.

### Main Module

This module is responsible for detecting the type of service module to
be used and calls the appropriate service module. A simple string search
for the service name is done on the input URL to find the type of
service. Errors are handled accordingly.

### Hulu

We first require the page source of the video.  The function
createSoupObject() is responsible for this. For this purpose we use the
requests module. We parse the HTML with the help of BeautifulSoup
library. The getTitle function returns the title of the video. This is
also used for naming the file. The title is present in the Soup Object.
Example -

<meta name="twitter:title" value="Interstellar"/>
 We then require the contentID of the video. This is also available
in the HTML Source.  This is one of the methodologies to get the
content ID. If this fails the alternative method will be called. In
the Beautiful soup text it can be found that every video has this
parameter.

` "content_id": "60535322"`

So we first use '"'(quotes) as the delimiter and split the text. Then
access the content ID from the returned list. The function
getSmiSubtitlesLink returns the SMI subtitle link based on the
contentID. The XML Link for any subtitle video is :

` `[`http://www.hulu.com/captions.xml?content_id=CONTENTID`](http://www.hulu.com/captions.xml?content_id=CONTENTID)

If multiple languages are present we give the user an option to enter
their choice. We then convert the SMI URL to a VTT URL as follows -
<http://assets.huluim.com/captions/380/60601380_US_en_en.smi> \-\--\>
<http://assets.huluim.com/captions_webvtt/380/60601380_US_en_en.vtt>
 Then the subtitles are converted from VTT to SRT format in the
standard way. 

### YouTube

We first require the page source of the video.  The function
createSoupObject() is responsible for this. For this purpose we use the
requests module. We parse the HTML with the help of BeautifulSoup
library. The getTitle function returns the title of the video. This is
also used for naming the file.

<title>
VIDEO NAME - YouTube

</title>
The function getRawSubtitleLink returns the Raw Link which is in encoded
format. This is still an incomplete URL. The variable UglyString
contains the complete URL. The link is present in the BeautifulSoup. We
now prompt the user to choose the desired language from the available
choices. The available subtitle language choices are extracted from the
UglyString. Based on the chosen language, the corresponding language
code is indexed from the language dictionary. This language code is
appended to the decoded Link.  This final URL contains the subtitles
as an XML file. Now, the XML file is converted to .srt file using
BeautifulSoup function calls. 

### Amazon

The subtitle URL for Amazon is present in this URL -

"PreURL":"<https://atv-ps.amazon.com/cdp/catalog/GetPlaybackResources>?",

`           "asin"                              : "" ,`\
`           "consumptionType"                   : "Streaming" ,`\
`           "desiredResources"                  : "SubtitleUrls" ,`\
`           "deviceID"                          : "b63345bc3fccf7275dcad0cf7f683a8f" ,`\
`           "deviceTypeID"                      : "AOAGZA014O5RE" ,`\
`           "firmware"                          : "1" ,`\
`           "marketplaceID"                     : "ATVPDKIKX0DER" ,`\
`           "resourceUsage"                     : "ImmediateConsumption" ,`\
`           "videoMaterialType"                 : "Feature" ,`\
`           "operatingSystemName"               : "Linux" ,`\
`           "customerID"                        : "" ,`\
`           "token"                             : "" ,`\
`           "deviceDrmOverride"                 : "CENC" ,`\
`           "deviceStreamingTechnologyOverride" : "DASH" ,`\
`           "deviceProtocolOverride"            : "Https" ,`\
`           "deviceBitrateAdaptationsOverride"  : "CVBR,CBR" ,`\
`           "titleDecorationScheme"             : "primary-content"`

 The primary parameters we need to get are ASIN ID, customerID and
TOKEN. These are obtained from the config file.  The config file is
generated from the setup.py file. The setup.py file takes the users
login and password and generates the config file. The ASINID is taken
from the URL directly.

` `[`https://www.amazon.com/dp/B019DSWVYC/?autoplay=1`](https://www.amazon.com/dp/B019DSWVYC/?autoplay=1)

Now, add the parameters to the dictionary and generate the final URL.
The final URL will look something like this -

` `[`https://atv-ps.amazon.com/cdp/catalog/GetPlaybackResources?&consumptionType=Streaming&titleDecorationScheme=primary-content&firmware=1&marketplaceID=ATVPDKIKX0DER&resourceUsage=ImmediateConsumption&deviceTypeID=AOAGZA014O5RE&videoMaterialType=Feature&token=6463643hhhdfhdhf7374747&deviceBitrateAdaptationsOverride=CVBR,CBR&operatingSystemName=Linux&deviceProtocolOverride=Https&deviceID=b63345bc3fccf7275dcad0cf7f683a8f&deviceStreamingTechnologyOverride=DASH&asin=B0141BACGU&desiredResources=SubtitleUrls&customerID=A1234GH2343&deviceDrmOverride=CENC`](https://atv-ps.amazon.com/cdp/catalog/GetPlaybackResources?&consumptionType=Streaming&titleDecorationScheme=primary-content&firmware=1&marketplaceID=ATVPDKIKX0DER&resourceUsage=ImmediateConsumption&deviceTypeID=AOAGZA014O5RE&videoMaterialType=Feature&token=6463643hhhdfhdhf7374747&deviceBitrateAdaptationsOverride=CVBR,CBR&operatingSystemName=Linux&deviceProtocolOverride=Https&deviceID=b63345bc3fccf7275dcad0cf7f683a8f&deviceStreamingTechnologyOverride=DASH&asin=B0141BACGU&desiredResources=SubtitleUrls&customerID=A1234GH2343&deviceDrmOverride=CENC)\
` `

This is where the Subtitle URL is present. We get a JSON response from
this URL and it contains a subtitle URL with .dfxp format. We request
that subtitle URL and download the subtitles.  With BeautifulSoup
and Python regex we convert this dfxp to .srt format. (File -
Amazon\_XmlToSrt.py) 

### BBC

We first need to extract the episode ID from the URL. Sample URL -

` `[`http://www.bbc.co.uk/iplayer/episode/p03rkqcv/shakespeare-lives-the-works`](http://www.bbc.co.uk/iplayer/episode/p03rkqcv/shakespeare-lives-the-works)\
` `

The episode ID is p03rkqcv. The episode PID and episode Title(for
naming the file) are present in the URL -

` `[`http://www.bbc.co.uk/programmes/`](http://www.bbc.co.uk/programmes/)<episode_id>`.xml`

 The subtitle URL is present in the following link -

` `[`http://open.live.bbc.co.uk/mediaselector/5/select/version/2.0/mediaset/pc/vpid/`](http://open.live.bbc.co.uk/mediaselector/5/select/version/2.0/mediaset/pc/vpid/)<pid>

The PID is nothing but the episode PID obtained above. There are
multiple PID's present. So, we try all the URL's until the page
request is successful.  If the request is successful we get the
subtitle link by parsing the XML page using Beautiful Soup. The
subtitles obtained are in XML format. They are converted to .srt by
using BeautifulSoup function calls and regex. The conversion takes place
in the file Bbc\_XmlToSrt.py 

### CrunchyRoll

This is one of the methodologies to get the subtitles ID. In the
Beautiful soup text it can be found that every video has this parameter.
<code>

<div>
Subtitles:

`         `<span class="showmedia-subtitle-text">\
`           `<img src="http://static.ak.crunchyroll.com/i/country_flags/us.gif"/>` `\
`           `<a href="/naruto-shippuden/episode-464-ninshu-the-ninja-creed-696237?ssid=206027" title="English (US)">`English (US)`</a>`,`\
`           `<img src="http://static.ak.crunchyroll.com/i/country_flags/sa.gif"/>` `\
`           `<a href="/naruto-shippuden/episode-464-ninshu-the-ninja-creed-696237?ssid=206015" title="العربية">`العربية`</a>`,`\
`           `<img src="http://static.ak.crunchyroll.com/i/country_flags/it.gif"/>` `\
`           `<a href="/naruto-shippuden/episode-464-ninshu-the-ninja-creed-696237?ssid=206733" title="Italiano">`Italiano`</a>`, `\
`           `<img src="http://static.ak.crunchyroll.com/i/country_flags/de.gif"/>\
`           `<a href="/naruto-shippuden/episode-464-ninshu-the-ninja-creed-696237?ssid=206033" title="Deutsch">`Deutsch`</a>\
`         `</span>\
`       `

</div>
~~~ We need to obtain all the SSID's. We return all the id's as a
list along with the respective Language title attached.  For the
above HTML we should have this - \[\['206027', 'English (US)'\],
\['206015', 'العربية'\], \['206733', 'Italiano'\], \['206033',
'Deutsch'\]\]  We prompt the user to choose the language and based
on the choice, we append the ID from the list obtained above. A sample
subtitle URL, where a script\_id(206027) has been appended to the base
URL :

` `[`http://www.crunchyroll.com/xml/?req=RpcApiSubtitle_GetXml&subtitle_script_id=206027`](http://www.crunchyroll.com/xml/?req=RpcApiSubtitle_GetXml&subtitle_script_id=206027)\
` `

The encrypted subtitles are extracted from the above URL. The decryption
of these subtitles has been taken from another Open Source software :
youtube-dl.

### Netflix

The user needs to input his username and password of Netflix in the
userconfig.ini file. Netflix requires login to download the subtitles.


We use python-selenium browser to automate the process. The first step
is to login to Netflix with the config file information. Chrome
WebDriver is used as the driver for selenium.  After a successful
login from selenium browser, we request for the video URL.  The
chrome Network tab gives a list of resources fetched from the server. We
use the command :

` return window.performance.getEntries();`

This command returns all the fetched URL's. It was observed that all
the Netflix videos had this sub-string in common and it was unique.
**/?o**  So we query for **/?o** and let the browser fetch
the resources until we find such a URL. If we do not find the URL before
the time out, we exit the application. If such a URL is found we save
the URL and follow the standard procedure.  We request the URL using
requests module and save the file.  The module
//Netflix\_XmlToSrt.py// is used to convert XML to .srt format.

### FOX

We first require the page source of the video.  The function
createSoupObject() is responsible for this. For this purpose we use the
requests module. We parse the HTML with the help of BeautifulSoup
library.

The video URL follows a specific standard throughout.
[`http://www.fox.com/watch/684171331973/7684520448`](http://www.fox.com/watch/684171331973/7684520448)
We need to split and return "684171331973". This is the required
contentID. 

This is the alternative method to obtain the contentID. In the soup text
there is a meta tag which also contains the video URL. This is helpful
in case the user inputs a shortened URL.

<code>

<meta content="http://www.fox.com/watch/684171331973/7684520448" property="og:url"/>
~~~ As stated above we split the URL and return the require
contentID, //684171331973// The other parameters required for obtaining
the subtitle URL are also present in the HTML page source.

The required script content looks like this- <code>

`       jQuery.extend(Drupal.settings, {"":...............}); `

~~~

` *We add everything to a new string after encountering the first "{".`\
` *Remove the last parentheses and the semi-colon to create a valid JSON. ---- ');'`

 The JSON has the standard format and the required parameters follow
this naming. The json content :
`{"foxProfileContinueWatching":{"showid":"empire","showname":"Empire"},..............`
`"foxAdobePassProvider": {......,"videoGUID":"2AYB18"}}`

We use the json module to parse the json and extract the parameters
namely //showid// , //showname// , //videoGUID// 

Sample Subtitle Links -

` `[`http://static-media.fox.com/cc/sleepy-hollow/SleepyHollow_3AWL18_660599363942.srt`](http://static-media.fox.com/cc/sleepy-hollow/SleepyHollow_3AWL18_660599363942.srt)\
` `[`http://static-media.fox.com/cc/sleepy-hollow/SleepyHollow_3AWL18_660599363942.dfxp`](http://static-media.fox.com/cc/sleepy-hollow/SleepyHollow_3AWL18_660599363942.dfxp)

The standard followed is -

` `[`http://static-media.fox.com/cc/[showid]/showname_videoGUID_contentID.srt`](http://static-media.fox.com/cc/%5Bshowid%5D/showname_videoGUID_contentID.srt)\
` `[`http://static-media.fox.com/cc/[showid]/showname_videoGUID_contentID.dfxp`](http://static-media.fox.com/cc/%5Bshowid%5D/showname_videoGUID_contentID.dfxp)

Some Subtitle URL's follow this standard -

` `[`http://static-media.fox.com/cc/[showid]/showname_videoGUID.dfxp`](http://static-media.fox.com/cc/%5Bshowid%5D/showname_videoGUID.dfxp)\
` `[`http://static-media.fox.com/cc/[showid]/showname_videoGUID.srt`](http://static-media.fox.com/cc/%5Bshowid%5D/showname_videoGUID.srt)

So we store both URL's and check for both the varieties. We request
both the varieties of URL and save the subtitles file when a successful
request is returned.

### General rules

Each service has a unique way of fetching the subtitles from the server.
We can get to know the methodology by following some steps -

` *The easiest way is to first open the Developer tools in Chrome/Firefox and check for XHR requests. Generally we find the subtitle URL's here.`\
` *The next step is to find out a general pattern in the subtitle URL's of that particular service.`\
` *If a pattern is found, it is most likely that we can request the subtitle page by forming the URL's from the required parameters. `\
` *Generally, the parameters can be found in the HTML page source. We need to search for them and query the URL.`\
` *Sometimes the required parameters for the URL are found in some other links in JSON format. A quick check of the fetched JSON resources will reveal the availability of them.`\
` *For services such as Netflix, the parameters have some kind of hashing in them which is difficult to decrypt. In such cases we can use selenium browser and search for keywords like **.srt**, **.dfxp**, **cc**, **sub**`\
` *By checking for multiple videos we can find out common sub-strings in the subtitle URLs. These common sub-strings(have to be unique) can be used for querying the resources from selenium browser.`\
` *In most cases, the subtitle URL is fetched only if the user is logged in. So we first need to setup login and then go to the video URL in the WebDriver.`\
` *The subtitles can then be downloaded from the URLs.    `

If you are a developer and want to add support for new services or fix bugs please feel free to send a pull request or contact me for further assistance.
---------------------------------------------------------------------------------------------------------------------------------------------------------
