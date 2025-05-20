import requests, random, re, hashlib, string
from time import sleep
import time

timestamp = str(int(time.time()))
uu = '83f2000a-4b95-4811-bc8d-0f3539ef07cf'
SESSION_ID = None  # متغير لحفظ session id

def RandomString(n=10):
    letters = string.ascii_lowercase + '1234567890'
    return ''.join(random.choice(letters) for _ in range(n))

def RandomStringChars(n=10):
    letters = string.ascii_lowercase
    return ''.join(random.choice(letters) for _ in range(n))

def randomStringWithChar(stringLength=10):
    letters = string.ascii_lowercase + '1234567890'
    result = ''.join(random.choice(letters) for _ in range(stringLength - 1))
    return RandomStringChars(1) + result

def generate_DeviceId(ID):
    volatile_ID = "12345"
    m = hashlib.md5()
    m.update(ID.encode('utf-8') + volatile_ID.encode('utf-8'))
    return 'android-' + m.hexdigest()[:16]

def generateUSER_AGENT():
    Devices_menu = ['HUAWEI', 'Xiaomi', 'samsung', 'OnePlus']
    DPIs = ['480', '320', '640', '515', '120', '160', '240', '800']
    randResolution = random.randrange(2, 9) * 180
    lowerResolution = randResolution - 180
    DEVICE_SETTINGS = {
        'system': "Android",
        'Host': "Instagram",
        'manufacturer': random.choice(Devices_menu),
        'model': f'{random.choice(Devices_menu)}-{randomStringWithChar(4).upper()}',
        'android_version': random.randint(18, 25),
        'android_release': f'{random.randint(1, 7)}.{random.randint(0, 7)}',
        "cpu": f"{RandomStringChars(2)}{random.randrange(1000, 9999)}",
        'resolution': f'{randResolution}x{lowerResolution}',
        'randomL': RandomString(6),
        'dpi': random.choice(DPIs)
    }
    return '{Host} 155.0.0.37.107 {system} ({android_version}/{android_release}; {dpi}dpi; {resolution}; {manufacturer}; {model}; {cpu}; {randomL}; en_US)'.format(
        **DEVICE_SETTINGS)

def headers_login(user_agent):
    return {
        'User-Agent': user_agent,
        'Host': 'i.instagram.com',
        'content-type': 'application/x-www-form-urlencoded; charset=UTF-8',
        'accept-encoding': 'gzip, deflate',
        'x-fb-http-engine': 'Liger',
        'Connection': 'close'
    }

def checkpoint(req, cookies, user_agent, username):
    info = requests.get("https://i.instagram.com/api/v1" + req.json()['challenge']['api_path'], headers=headers_login(user_agent), cookies=cookies)
    step_data = info.json().get("step_data", {})
    if "phone_number" in step_data:
        print(f'[0] phone_number : {step_data["phone_number"]}')
    elif "email" in step_data:
        print(f'[1] email : {step_data["email"]}')
    else:
        print("Unknown verification method.")
        return

    choice = input('Choice: ')
    data = {
        'choice': str(choice),
        '_uuid': uu,
        '_uid': uu,
        '_csrftoken': 'massing'
    }
    challnge = req.json()['challenge']['api_path']
    send = requests.post(f"https://i.instagram.com/api/v1{challnge}", headers=headers_login(user_agent), data=data, cookies=cookies)
    contact_point = send.json()["step_data"]["contact_point"]
    print(f'Code sent to: {contact_point}')
    
    return get_code(req, cookies, user_agent, username)

def get_code(req, cookies, user_agent, username):
    global SESSION_ID
    code = input("Code: ")
    data = {
        'security_code': code,
        '_uuid': uu,
        '_uid': uu,
        '_csrftoken': 'massing'
    }
    path = req.json()['challenge']['api_path']
    send_code = requests.post(f"https://i.instagram.com/api/v1{path}", headers=headers_login(user_agent), data=data, cookies=cookies)
    
    if "logged_in_user" in send_code.text:
        print(f'Login Successfully as @{username}')
        SESSION_ID = send_code.cookies.get("sessionid")
        print("Session ID:", SESSION_ID)
    else:
        try:
            regx_error = re.search(r'"message":"(.*?)",', send_code.text).group(1)
            print(regx_error)
        except:
            print(send_code.text)
        ask = input("Code is not working. Try again? [Y/N]: ")
        if ask.lower() == "y":
            return get_code(req, cookies, user_agent, username)
        else:
            exit()

def login():
    global SESSION_ID
    username = input('Username: ')
    password = input('Password: ')
    user_agent = generateUSER_AGENT()
    device_id = generate_DeviceId(username)

    data = {
        'guid': uu,
        'enc_password': f"#PWD_INSTAGRAM:0:{timestamp}:{password}",
        'username': username,
        'device_id': device_id,
        'login_attempt_count': '0'
    }

    req = requests.post("https://i.instagram.com/api/v1/accounts/login/", headers=headers_login(user_agent), data=data)

    if "logged_in_user" in req.text:
        print(f'Login Successfully as @{username}')
        SESSION_ID = req.cookies.get("sessionid")
        print("Session ID:", SESSION_ID)
    elif 'checkpoint_challenge_required' in req.text:
        print("Challenge checkpoint required.")
        return checkpoint(req, req.cookies, user_agent, username)
    else:
        try:
            regx_error = re.search(r'"message":"(.*?)",', req.text).group(1)
            print(regx_error)
        except:
            print(req.text)
        ask = input("Something went wrong. Try again? [Y/N]: ")
        if ask.lower() == "y":
            return login()
        else:
            exit()

login()
