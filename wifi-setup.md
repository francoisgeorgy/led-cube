
## Tool

    sudo nmtui

Edit a connection

    <Add>
    Wi-Fi
    set : 
        Profile name : 
        SSID : 
        Mode : <Client>
        Security : WPA & WPA2 Personal
        Password : 
    <Back>
    Quit
    <OK>

## Files

The networks definitions files are in `/etc/NetworkManager/system-connections/`.

## See also : 

- https://forums.raspberrypi.com/viewtopic.php?t=360175
- https://forums.raspberrypi.com/viewtopic.php?t=357623

## useful commands
    
    # Quick status of all interfaces
    nmcli dev status
    
    # Detailed overview of all interfaces
    nmcli
    
    # Get a list of all WiFi SSIDs
    nmcli d wifi list
    
    # Disconnect from a WiFi network
    nmcli con  # Get the NAME of the WiFi connection
    nmcli con down "connection_name_here"
    
    # Connect to a WiFi network on a specific WiFi interface
    sudo nmcli d wifi connect "ssid_here" password "password_here" ifname wlan1
    
    # Show detailed information about a specific WiFi interface
    iw dev wlan0 info
    
    # Show WiFi link details (signal strength, bitrates)
    iw dev wlan0 link
