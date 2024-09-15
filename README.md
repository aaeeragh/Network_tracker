# Network_tracker
Building a network tracker using Python , Wireshark and Google Maps 

This project is to visualization Network Traffic using the Python programming language, Wireshark and Google Maps.

###Step-1#####
GeoLite Download 
First of you will need to download the GeoLiteCity database as this will be used to translate a IP address into a Geo location(longitude & latitude)
Link - https://github.com/mbcc2006/GeoLiteCity-data

###Step-2###
Wireshark 
Besides the GeoLiteCity database we also be needing the Wireshark application to be able to capture network traffic in our device. We will study and monitor the network hops of our device and we will collect the hoping infos through wireshark . 
- just go into wireshark , click your network connection
- an instance of hops will start .
- close in withing 2 seconds of enabling it as we do not have to analyze all the hops . Just few will be fine
- 
- save the file as shown below


###Step-3###
Python Implementation

Packages 

```
import dpkt
import socket
import pygeoip

gi = pygeoip.GeoIP('GeoLiteCity.dat')
```

Next off you will be implementing the main method, which will open your captured data along with creating the header and footer of the KML file, which is the output file that we will upload to Google Maps. Again be sure to place you captured .pcap file in the root of the project and change the name of the code below if you did not save it under the name[yourfilename.pcap]

```
def main():
    f = open('[yourfilename].pcap', 'rb') '''add your wireshark saved filename'''
    pcap = dpkt.pcap.Reader(f)
    kmlheader = '<?xml version="1.0" encoding="UTF-8"?> \n<kml xmlns="http://www.opengis.net/kml/2.2">\n<Document>\n'\
    '<Style id="transBluePoly">' \
                '<LineStyle>' \
                '<width>1.5</width>' \
                '<color>501400E6</color>' \
                '</LineStyle>' \
                '</Style>'
    kmlfooter = '</Document>\n</kml>\n'
    kmldoc=kmlheader+plotIPs(pcap)+kmlfooter
    print(kmldoc)
```

Next of we’re to add the method that will loop over our captured network data and extract the IP’s.

```
def plotIPs(pcap):
    kmlPts = ''
    for (ts, buf) in pcap:
        try:
            eth = dpkt.ethernet.Ethernet(buf)
            ip = eth.data
            src = socket.inet_ntoa(ip.src)
            dst = socket.inet_ntoa(ip.dst)
            KML = retKML(dst, src)
            kmlPts = kmlPts + KML
        except:
            pass
    return kmlPts
```
We need to attach a Geo location. This will be done using the following method-

```
def retKML(dstip, srcip):
    dst = gi.record_by_name(dstip)
    src = gi.record_by_name('x.xxx.xxx.xxx')  ''' Enter your ip '''
    try:
        dstlongitude = dst['longitude']
        dstlatitude = dst['latitude']
        srclongitude = src['longitude']
        srclatitude = src['latitude']
        kml = (
            '<Placemark>\n'
            '<name>%s</name>\n'
            '<extrude>1</extrude>\n'
            '<tessellate>1</tessellate>\n'
            '<styleUrl>#transBluePoly</styleUrl>\n'
            '<LineString>\n'
            '<coordinates>%6f,%6f\n%6f,%6f</coordinates>\n'
            '</LineString>\n'
            '</Placemark>\n'
        )%(dstip, dstlongitude, dstlatitude, srclongitude, srclatitude)
        return kml
    except:
        return ''
```

In order to know your Public IP you can use - https://www.whatsmyip.org/

Execute the above code with main function -

```
if __name__ == '__main__':
    main()
```
Run the script to get the .knl file with details of latitute , longitude and hops that can be compatible with google maps 

###Step-4###
Adding Data to Google Maps

Open google maps and add the newly created .kml file which was formed after running the python script .
Click on Import and then select the .kml file on your computer, once the file is uploaded you should see the network traffic being dislayed on the map.
