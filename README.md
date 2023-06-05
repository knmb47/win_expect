# win_expect
Ansible module `win_expect`

This is a placeholder repository to publish ansible win_expect module, which is already developed and shared internally in kyndryl.

If you have needs to use `win_expect` right now, please approach any of your kyndryl contact including kunio.namba@kyndryl.com .
The `win_expect` module can work as like `expect` ansible module for UNIX. It will work on Windows OS with ansible using attached example code.
When the conversation with target command was made without problem, it will finish successfully and the result will have the output message of the conversation.

```
#   Copyright 2023 Kunio Namba
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
---
- hosts: windows
  tasks:
          - name: call windows shell
            win_shell: |
                    echo test
            register: cmd_result

          - name: dump the shell output
            debug: var=cmd_result

          - name: call win_expect
            kyndryl_ansible.win_expect_collection.win_expect:
                    command: cmd.exe
                    command_args: '/c "set /p var1=test: && set var1"'
                    expect_rules: |
                        [
                                {
                                        "expect": "test",
                                        "send": "avalue",
                                        "remarks": "step1"
                                },
                                {
                                        "expect": ".*avalue",
                                        "remarks": "step2"
                                }
                        ]
                    debug: True
            register: cmd_result

          - name: dump the win_expect output
            debug: var=cmd_result
```


another example case to manipulate operation menu type of interface on DOS.
```
#   Copyright 2023 Kunio Namba
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
- hosts: windows
  tasks:
          - name: call windows shell
            win_shell: |
                    chcp 932
                    chcp
            register: cmd_result

          - name: dump the shell output
            debug: var=cmd_result

          - name: call win_expect
            kyndryl_ansible.win_expect_collection.win_expect:
                    command: cmd.exe
                    command_args: '/c "chcp 932 && C:/temp/menu.bat"'
                    debug: true
                    codepage: "932"
                    encoding: Shift-JIS
                    expect_rules: |
                        [
                                {
                                        "read": 1024,
                                        "interval": 500,
                                        "interval_count": 3,
                                        "expect": "オペレーション・メニュー・サンプル[\\s\\S]*Main Menu[\\s\\S]*pingを実行[\\s\\S]*希望のコマンド番号を入力してください",
                                        "remarks": "Menu Text was not shown in step 1",
                                        "send": "1"
                                },
                                {
                                        "read": 1024,
                                        "interval": 3000,
                                        "interval_count": 3,
                                        "expect": "Ping先のホスト名",
                                        "remarks": "Host was not asked for ping",
                                        "send": "127.0.0.1"
                                },
                                {
                                        "read": 1024,
                                        "interval": 5000,
                                        "interval_count": 5,
                                        "expect": "Ping実行します。[\\s\\S]*確認したらEnterキーを押してください",
                                        "remarks": "Ping result confirm message was not shown",
                                        "send": ""
                                },
                                {
                                        "interval": 1000,
                                        "interval_count": 3,
                                        "expect": "オペレーション・メニュー・サンプル[\\s\\S]*Main Menu[\\s\\S]*ipconfigを実行[\\s\\S]*希望のコマンド番号を入力してください",
                                        "remarks": "Menu Text was not shown in step 4",
                                        "send": "2"
                                },
                                {
                                        "interval": 1000,
                                        "interval_count": 3,
                                        "expect": "ipconfig実行します。[\\s\\S]*確認したらEnterキーを押してください",
                                        "remarks": "ipconfig result confirm message was not shown",
                                        "send": ""
                                },
                                {
                                        "interval": 1000,
                                        "interval_count": 3,
                                        "expect": "希望のコマンド番号を入力してください",
                                        "remarks": "Menu Text was not shown in step 6",
                                        "send": "q"
                                },
                                {
                                        "read": 4096,
                                        "interval": 1000,
                                        "interval_count": 1,
                                        "expect": ".*",
                                        "remarks": "Unexpected situation when consuming final menu messages."
                                }
                        ]
            register: cmd_result

          - name: dump the win_expect output
            debug: var=cmd_result
```
