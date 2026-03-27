---
title: '[AI for Security] Nmap MCP로 서버 스캔하기 with Claude'
date: 2026-03-27 00:00:00 +0900
last_modified_at: 2026-03-27
categories: [Pentest]
tags: []
published: True
---

# #Intro

개발에 관심을 가지고 공부하기 시작한 이후로 꾸준히 velog 인기글을 살펴보곤 했는데, 작년부터인가 블로그 읽을 시간을 내지 않았고 최근 인턴으로 출퇴근하며 다시 들어간 velog에는 재미있는 양질의 글들이 쌓여있었다. 그 중에서도 올해의 트렌딩 글로 218개의 좋아요를 받은 글이 있었는데, [MCP의 모든 것을 알아봅시다](https://velog.io/@k-svelte-master/what-is-mcp) 글이었다. MCP에 대해서는 1년 동안 너무나 많이 접해왔기에 MCP 라는 것이 무엇이고 어떻게 활용되는지는 모를 수가 없었지만 MCP가 어떻게 동작하는지에 대한 공부는 미루고 있었고, 때마침 이 글을 마주하게 되었다. 그래서 이 블로그 글을 계기로 직접 부딪혀보면서 MCP를 적극 활용해보고, 이 과정들을 정리하여 기록하고자 한다.

<br>

# #Set-up

가장 먼저, 프로젝트 초기 설정을 진행한다.

### 1. 필요한 도구 설치

- Python 및 pip
- [Nmap](https://nmap.org/download.html)
- [Claude Desktop](https://claude.com/download)

<br>

### 2. 프로젝트 폴더 생성

```powershell
mkdir C:\\Users\\r4m\\r4m-mcp-server
```

<br>

### 3. MCP Python SDK 설치

```powershell
pip install mcp
```

<br>

### 4. MCP server 파일 생성

프로젝트 폴더 내에 `nmap-mcp-server.py` 파일을 생성하고, 아래와 같이 MCP 서버 코드를 작성한다.

```python
import asyncio
import json
import subprocess
from typing import Any, Optional
import xml.etree.ElementTree as ET

from mcp.server import Server
from mcp.types import Tool, TextContent, ImageContent, EmbeddedResource

# MCP 서버 초기화
app = Server("nmap-server")

# 도구 목록 정의
@app.list_tools()
async def list_tools() -> list[Tool]:
    """Claude가 사용할 수 있는 nmap 도구 목록"""
    return [
        Tool(
            name="nmap_port_scan",
            description="대상 호스트의 포트를 스캔합니다. 기본적으로 상위 1000개 포트를 스캔합니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "스캔할 대상 (IP 주소 또는 도메인)"
                    },
                    "ports": {
                        "type": "string",
                        "description": "스캔할 포트 범위 (예: '80,443' or '1-1000')",
                        "default": ""
                    },
                    "scan_type": {
                        "type": "string",
                        "enum": ["tcp", "syn", "udp"],
                        "description": "스캔 타입: tcp(연결 스캔), syn(SYN 스캔), udp(UDP 스캔)",
                        "default": "tcp"
                    }
                },
                "required": ["target"]
            }
        ),
        Tool(
            name="nmap_service_detection",
            description="열린 포트에서 실행 중인 서비스와 버전을 감지합니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "스캔할 대상"
                    },
                    "ports": {
                        "type": "string",
                        "description": "스캔할 포트 범위",
                        "default": ""
                    }
                },
                "required": ["target"]
            }
        ),
        Tool(
            name="nmap_os_detection",
            description="대상 시스템의 운영체제를 감지합니다. (관리자 권한 필요)",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "스캔할 대상"
                    }
                },
                "required": ["target"]
            }
        ),
        Tool(
            name="nmap_script_scan",
            description="NSE(Nmap Scripting Engine) 스크립트를 실행합니다.",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "스캔할 대상"
                    },
                    "script": {
                        "type": "string",
                        "description": "실행할 스크립트 (예: 'vuln', 'default', 'http-title')",
                        "default": "default"
                    },
                    "ports": {
                        "type": "string",
                        "description": "스캔할 포트",
                        "default": ""
                    }
                },
                "required": ["target"]
            }
        ),
        Tool(
            name="nmap_ping_scan",
            description="대상 호스트가 활성화되어 있는지 확인합니다 (포트 스캔 없음).",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "확인할 대상 (단일 IP 또는 네트워크 범위)"
                    }
                },
                "required": ["target"]
            }
        )
    ]


def run_nmap_command(args: list[str], timeout: int = 300) -> dict:
    """nmap 명령을 실행하고 결과를 파싱"""
    try:
        # XML 출력 옵션 추가
        full_args = ["nmap", "-oX", "-"] + args

        result = subprocess.run(
            full_args,
            capture_output=True,
            text=True,
            timeout=timeout,
            check=False
        )

        if result.returncode != 0 and result.returncode != 1:
            # nmap은 호스트가 다운되어도 returncode 1을 반환할 수 있음
            return {
                "success": False,
                "error": result.stderr or "Unknown error",
                "raw_output": result.stdout
            }

        # XML 파싱
        if result.stdout:
            return parse_nmap_xml(result.stdout)
        else:
            return {
                "success": False,
                "error": "No output from nmap",
                "raw_output": result.stderr
            }

    except subprocess.TimeoutExpired:
        return {
            "success": False,
            "error": f"Scan timed out after {timeout} seconds"
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }


def parse_nmap_xml(xml_output: str) -> dict:
    """nmap XML 출력을 파싱하여 구조화된 데이터로 변환"""
    try:
        root = ET.fromstring(xml_output)

        result = {
            "success": True,
            "scan_info": {},
            "hosts": []
        }

        # 스캔 정보
        runstats = root.find("runstats")
        if runstats is not None:
            finished = runstats.find("finished")
            if finished is not None:
                result["scan_info"]["elapsed"] = finished.get("elapsed", "")
                result["scan_info"]["summary"] = finished.get("summary", "")

        # 호스트 정보
        for host in root.findall("host"):
            host_info = {
                "status": host.find("status").get("state") if host.find("status") is not None else "unknown",
                "addresses": [],
                "hostnames": [],
                "ports": [],
                "os": {}
            }

            # IP 주소
            for addr in host.findall("address"):
                host_info["addresses"].append({
                    "addr": addr.get("addr"),
                    "type": addr.get("addrtype")
                })

            # 호스트명
            hostnames = host.find("hostnames")
            if hostnames is not None:
                for hostname in hostnames.findall("hostname"):
                    host_info["hostnames"].append({
                        "name": hostname.get("name"),
                        "type": hostname.get("type")
                    })

            # 포트 정보
            ports = host.find("ports")
            if ports is not None:
                for port in ports.findall("port"):
                    port_info = {
                        "port": port.get("portid"),
                        "protocol": port.get("protocol"),
                        "state": port.find("state").get("state") if port.find("state") is not None else "unknown",
                        "service": {}
                    }

                    service = port.find("service")
                    if service is not None:
                        port_info["service"] = {
                            "name": service.get("name", ""),
                            "product": service.get("product", ""),
                            "version": service.get("version", ""),
                            "extrainfo": service.get("extrainfo", "")
                        }

                    # 스크립트 결과
                    scripts = port.findall("script")
                    if scripts:
                        port_info["scripts"] = []
                        for script in scripts:
                            port_info["scripts"].append({
                                "id": script.get("id"),
                                "output": script.get("output")
                            })

                    host_info["ports"].append(port_info)

            # OS 감지
            os_elem = host.find("os")
            if os_elem is not None:
                osmatch = os_elem.find("osmatch")
                if osmatch is not None:
                    host_info["os"] = {
                        "name": osmatch.get("name"),
                        "accuracy": osmatch.get("accuracy")
                    }

            result["hosts"].append(host_info)

        return result

    except ET.ParseError as e:
        return {
            "success": False,
            "error": f"XML parsing error: {str(e)}",
            "raw_output": xml_output
        }


@app.call_tool()
async def call_tool(name: str, arguments: Any) -> list[TextContent]:
    """도구 호출 처리"""

    if name == "nmap_port_scan":
        target = arguments.get("target")
        ports = arguments.get("ports", "")
        scan_type = arguments.get("scan_type", "tcp")

        args = []
        if scan_type == "syn":
            args.append("-sS")
        elif scan_type == "udp":
            args.append("-sU")
        else:
            args.append("-sT")

        if ports:
            args.extend(["-p", ports])

        args.append(target)

        result = run_nmap_command(args)

    elif name == "nmap_service_detection":
        target = arguments.get("target")
        ports = arguments.get("ports", "")

        args = ["-sV"]
        if ports:
            args.extend(["-p", ports])
        args.append(target)

        result = run_nmap_command(args)

    elif name == "nmap_os_detection":
        target = arguments.get("target")
        args = ["-O", target]

        result = run_nmap_command(args)

    elif name == "nmap_script_scan":
        target = arguments.get("target")
        script = arguments.get("script", "default")
        ports = arguments.get("ports", "")

        args = ["--script", script]
        if ports:
            args.extend(["-p", ports])
        args.append(target)

        result = run_nmap_command(args, timeout=600)

    elif name == "nmap_ping_scan":
        target = arguments.get("target")
        args = ["-sn", target]

        result = run_nmap_command(args, timeout=60)

    else:
        result = {
            "success": False,
            "error": f"Unknown tool: {name}"
        }

    return [TextContent(
        type="text",
        text=json.dumps(result, indent=2, ensure_ascii=False)
    )]


async def main():
    """서버 실행"""
    from mcp.server.stdio import stdio_server

    async with stdio_server() as (read_stream, write_stream):
        await app.run(
            read_stream,
            write_stream,
            app.create_initialization_options()
        )


if __name__ == "__main__":
    asyncio.run(main())
```

[📎 source code 출처](https://hagsig.tistory.com/356#2-2.%20Nmap%20MCP%20Server%20%EC%84%A4%EC%B9%98%C2%A0-1-4)

<br>

### 5. Claude Desktop 연결

Claude Desktop은 MCP와 상호작용하기 위한 인터페이스로, Claude AI와의 통신을 원활하게 해준다.
Claude Desktop을 설치하고, MCP 서버 사용을 위해 설정 파일에 MCP 서버 정보를 입력해야 한다. Windows의 경우, 일반적으로 `C:\Users\사용자명\AppData\Roaming\Claude\claude_desktop_config.json` 경로에 설정 파일이 위치한다. 설정 파일을 열어 다음과 같이 MCP 서버 정보를 추가한다.

```json
{
  // 초기 파일 내용
  "preferences": {
    "legacyQuickEntryEnabled": true,
    "coworkScheduledTasksEnabled": false,
    "ccdScheduledTasksEnabled": false,
    "coworkWebSearchEnabled": true,
    "sidebarMode": "chat"
  },
  // MCP 서버 정보 추가
  "mcpServers": {
    "nmap": {
      "command": "python",
      "args": [
        "C:\\Users\\r4m\\r4m-mcp-server\\nmap-mcp-server.py"
      ],
      "env": {
        "PYTHONBUFFERED": "1"
      }
    }
  }
}
```

<br>


![image1](/assets/posts/260327-1.png)

이후 Claude Desktop을 재시작하면, MCP 서버가 자동으로 실행되고 Claude AI와 연결되어 위의 이미지와 같이 nmap이 커넥터에 등록된 것을 확인할 수 있다.

<br>

### 6. Target Server 준비

nmap MCP 서버 테스트를 위해서는 스캔 대상 서버가 필요하다. 테스트를 위해 Docker를 활용하여 metasploitable2 이미지를 실행한다.

<br>

1. Docker image pull
  ```powershell
docker pull tleemcjr/metasploitable2
  ```

2. Docker container 실행
  ```powershell
docker run -d -it `
--name metasploitable2 `
--hostname metasploitable2 `
-p 4444:80 `
--privileged `
tleemcjr/metasploitable2
  ```

<br>

![image3](/assets/posts/260327-3.png)

http://localhost:4444 주소로 접속하여 metasploitable2의 웹 서비스가 정상적으로 동작하는지 확인하면, 모든 준비가 완료되었다.

<br>

# #Scan with Claude

이제 모든 준비가 완료되었으니, Claude AI를 활용하여 nmap MCP 서버에 명령을 보내고 스캔을 수행해보자. Claude Desktop에서 새로운 채팅을 시작하고, 다음과 같이 입력한다.

```
nmap을 사용해서 http://localhost:4444 서버의 상위 1000개 포트를 TCP 스캔으로 스캔해줘.
```



<br>

<br>

> Reference
> - [https://hagsig.tistory.com/356](https://hagsig.tistory.com/356)
> - [https://wikidocs.net/book/17996](https://wikidocs.net/book/17996)
> - [https://velog.io/@k-svelte-master/what-is-mcp](https://velog.io/@k-svelte-master/what-is-mcp)
{: .prompt-info }
