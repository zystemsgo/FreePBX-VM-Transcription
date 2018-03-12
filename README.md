# FreePBX-VM-Transcription
Script to transcribe voicemail and convert voicemail recordings to MP3.

Forked and modify to use Google Speech API instead of Microsoft.

Also allows to filter to users with transcribevms=yes in description field.

IMPORTANT: This requires the exact phrase "There is a new voicemail in mailbox EXT_HERE:" to be in the body of the message.

## Installation

Need to install Google Python APIs
pip install --upgrade google-cloud-speech

Pull the repo to your server

    git clone https://github.com/westparkcom/FreePBX-VM-Transcription.git

Install the scripts:

    cd FreePBX-VM-Transcription
    cp emailproc /usr/local/bin
    cp sttparse /usr/local/bin
    chmod +x /usr/local/bin/emailproc
    chmod +x /usr/local/bin/sttparse

Create temporary directory for wav processing:

    mkdir /tmp/conv
    chmod 0777 /tmp/conv

Modify the "globalconfig" function parameters in the file /usr/local/bin/sttparse to suit your needs. The defaults are:

	'vm_transcription' : True, # Transcribe voicemail to text (requires Microsoft Azure Cognitive services subscription)
	'transcription_string' : '{{{{TRANSCRIPTION}}}}', # String to search for in the email which will be replaced by the transcribed text
	'gcloud_service_json' : '/path/to/your/service.json', # Gcloud JSON file location
	'speech_language' : 'en-US', # Language for speech recognition
	'vm_to_mp3' : False, # Convert voicemail audio to MP3
	'temp_dir': '/tmp/conv', # Temp location for MP3 conversion. MUST BE WRITABLE!
	'ffmpeg_location' : '/usr/bin/ffmpeg', # If vm_to_mp3 is True, location of ffmpeg executable
	'sox_location' : '/usr/bin/sox', # If vm_to_mp3 is True, location of ffmpeg executable
	'check_html' : True, # Check to see if message is HTML and set mimetype accordingly
	'check_descr_fortag' : True, # Check to see if user description contains string "transcribevms=yes"
	'db': 'asterisk', # your DB username
	'db_user': 'fpbxread', # your DB username
	'db_pass': 'mZn@Alk!', # your DB username
	'db_host': 'localhost', # your DB username

A Google Apps cloud services service API json is required for transcription and can be acquired from Google. Must enable Google Speech API

## FreePBX Setup

In FreePBX browse to **Settings -> Voicemail Admin -> Settings -> Email Config** then add the transcription tag **{{{{TRANSCRIPTION}}}}** to the **Email Body**. The script will search for this tag to replace with the transcription text. You can also use HTML and the script will automatically change the content type to text/html. Here is a sample html email:

    <html>
    <body>
    <p>${VM_NAME},
    <br><br>
    There is a new voicemail in mailbox ${VM_MAILBOX}:</p>
    <p>
    <table>
      <tr>
        <th align="left">From (Name):</th>
        <td>${VM_CIDNAME}</th>
      </tr>
      <tr>
        <th align="left">From (Number):</td>
        <td><a href="tel://${VM_CIDNUM}">${VM_CIDNUM}</a></td>
      </tr>
      <tr>
        <th align="left">Length:</td>
        <td>${VM_DUR} seconds</td>
      </tr>
      <tr>
        <th align="left">Date:</td>
        <td>${VM_DATE}</td>
      </tr>
      <tr>
        <th align="left">Transcription:</td>
        <td>{{{{TRANSCRIPTION}}}}</td>
      </tr>
    </table></p>
    <p>Dial *98 to access your voicemail by phone.<br>
    Visit <a href="https://your.pbxaddress.tld">https://your.pbxaddress.tld</a> to check your voicemail with a web browser.</p>
    </body>
    </html>

Finally, set the **Mail Command** value to **/usr/local/bin/emailproc**