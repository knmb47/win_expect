# win_expect
Ansible module `win_expect`

This is a placeholder repository to publish ansible win_expect module, which is already developed and shared internally in kyndryl.

If you have needs to use `win_expect` right now, please approach any of your kyndryl contact including kunio.namba@kyndryl.com .
The `win_expect` module can work as like `expect` ansible module for UNIX. It will work on Windows OS with ansible using attached example code.
When the conversation with target command was made without problem, it will finish successfully and the result will have the output message of the conversation.

---
   Copyright 2023 Kunio Namba

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

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
