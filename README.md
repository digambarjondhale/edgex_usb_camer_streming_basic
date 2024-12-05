**IMP NOTE**  :- Refrance https://docs.edgexfoundry.org/2.3/examples/Ch-ExamplesUsbDeviceService/

**Prerequisites**

    System Requirements:
        Linux environment with Docker and Docker Compose installed.
        A USB camera connected and accessible at /dev/video*.
        Internet access for downloading repositories and Docker images.

    Additional Tools:
        jq for parsing JSON responses.
        ffplay from FFmpeg for playing RTSP streams.

    References:
        EdgeX USB Device Service Example

Steps to Configure and Run
**Step 1: Setup EdgeX Environment**
mkdir ~/edgex
cd ~/edgex
git clone https://github.com/digambarjondhale/edgex_usb_camer_streming_basic.git

cd edgex_usb_camer_streming_basic
tar -xvf edgex-compose.tar.xz
cd ../
cp -r edgex_usb_camer_streming_basic/edgex-compose .
rm -rf edgex_usb_camer_streming_basic/

    cd edgex-compose/compose-builder

**Step 2: Launch EdgeX Services Run EdgeX with the USB camera device service in no-security mode:**

    sudo make run ds-usb-camera no-secty

**Step 3: Register the USB Camera Device**

    sudo curl -X POST -H 'Content-Type: application/json' \
    http://localhost:59881/api/v3/device \
    -d '[
      {                               
        "apiVersion": "v3",
        "device": {
          "name": "device-usb-camera",
          "serviceName": "device-usb-camera",
          "profileName": "USB-Camera-General",
          "description": "My test camera",
          "adminState": "UNLOCKED",
          "operatingState": "UP",
          "protocols": {
            "USB": {
              "CardName": "UVC Camera (046d:0825)",
              "Path": "/dev/video0",
              "AutoStreaming": "false",
              "Paths": ["/dev/video0"]
            }
          }
        }
      }
    ]'

**Step 4: Start the Camera Stream**
    curl -X PUT -d '{
      "StartStreaming": {
        "InputImageSize": "640x480",
        "OutputVideoQuality": "5"
      }
    }' http://localhost:59882/api/v3/device/name/device-usb-camera/StartStreaming

**Step 5: Retrieve the RTSP Stream URI**


curl -s http://localhost:59882/api/v3/device/name/device-usb-camera/StreamURI | jq -r '"StreamURI: " + .event.readings[].value'

**Example response**:

    StreamURI: rtsp://localhost:8554/stream/device-usb-camera

**Step 6: Play the Stream**

    ffplay rtsp://localhost:8554/stream/device-usb-camera


