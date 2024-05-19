
# day6 condition (条件分岐)

- condition(条件分岐)を学習する

- 目次
  - 1.condition(条件分岐)とは？
  - 2.whenディレクティブの使用例
  - 3.whenディレクティブの実習(ハンズオン)
  - condition(条件分岐)のまとめ
  - 4.whenディレクティブの演習問題

<br>
<br>
<br>

---

## 1. condition (条件分岐)とは？

- 条件分岐をさせたい時は、**Whenディレクティブ**を使用する。
- whenディレクティブを使用する例
  - 特定の条件に当てはまるタスクのみ実行したい時
  - 特定の条件に当てはまるタスクのみスキップしたい時 ...etc
- whenディレクティブを使用するときは、比較演算子・論理演算子・in演算子・is演算子の条件式を用いる。

- whenディレクティブのAnsible documentは[こちら](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html)

---

## 2. whenディレクティブの使用例

### 比較演算子でのwhenディレクティブの使用例

| 比較演算子 | trueになるとき|
| :-----: | :------------------------------------------------------------------------------------------------------------ |
| X == Y | X の値と Y の値が等しいとき |
| X != Y | X の値と Y の値が等しくないとき |
| X > Y | X の値が Y の値より大きいとき |
| X >= Y | X の値が Y の値より大きいか等しいとき |

- 以下は、実行対象ノードが「host01」だった場合、httpdをインストールするplaybookである。

```yaml
---
- name: Sample
  hosts: host01
  gather_facts: false

  tasks:
    - name: Sample1
      ansible.builtin.yum: 
        name: Httpd
        state: latest
      when: inventory_hostname == 'host01'
```

- yumモジュール・・・パッケージの管理（インストール、更新、削除など）をする

### 論理演算子でのwhenディレクティブの使用例

| 論理演算子 | trueになるとき|
| :-----: | :------------------------------------------------------------------------------------------------------------ |
| not 条件X | 条件X が Falseのとき |
| 条件X and 条件Y | 条件X と条件Y がともに True のとき |
| 条件X or 条件Y | 条件X か条件Y のどちらかが True のとき |

- 以下は、実行対象ノードが「host01」か「host02」だった場合、httpdをインストールするplaybookである。

```yaml
---
- name: Sample
  hosts: host01
  gather_facts: false

  tasks:
    - name: Sample2
      ansible.builtin.yum: 
        name: Httpd
        state: latest
      when: inventory_hostname == 'host01' or inventory_hostname == 'host02'
```

### in演算子でのwhenディレクティブの使用例

| in演算子 | trueになるとき|
| :-----: | :------------------------------------------------------------------------------------------------------------ |
| A in [X, Y, Z] | A と同じ値が X, Y, Z の中にあるとき |
| A not in [X, Y, Z] | A と同じ値が X, Y, Z の中にないとき |

- 以下は、実行対象ノードがリストに存在する「host01」か「host02」だった場合、httpdをインストールするplaybookである。

```yaml
---
- name: Sample
  hosts: host01
  gather_facts: false

  tasks:
    - name: Sample3
      ansible.builtin.yum: 
        name: Httpd
        state: latest
      when: inventory_hostname in ['host01','host02']
```

### is演算子でのwhenディレクティブの使用例

| is演算子 | trueになるとき|
| :-----: | :------------------------------------------------------------------------------------------------------------ |
| A is B | AがBの状態であるとき |
| A is not B | AがBの状態でないとき |

- 以下は、hostにhttpdインストールを実施し、httpdインストールが成功した場合「yum httpd succeess!!」、  
httpdインストールが失敗した場合「yum httpd error!!」というメッセージを出力するplaybookである
- httpdインストールが失敗してもplaybookが続行されるように「ignore_errors: true」を記述

```yaml
---
- name: Sample
  hosts: host
  gather_facts: false

  tasks:
    - name: Sample4
      ansible.builtin.yum: 
        name: Httpd
        state: latest
      register: result
      ignore_errors: true
      
    - name: Yum httpd success msg
      ansible.builtin.debug:
        msg: "yum httpd succeess!!"
      when: result is succeeded

    - name: Yum httpd error msg
      ansible.builtin.debug:
        msg: "yum httpd error!!"
      when: result is not succeeded
```

<br>
<br>
<br>

---

## 3. whenディレクティブの実習(ハンズオン)

### 目的

- vyos01に、「show ip route」「show interface」を実行させる(但しhosts: vyosとする)
- 実行結果を出力させる

### 1.ディレクトリ移動

- 使用するplaybook,inventoryファイルが存在するディレクトリに移動

```shell
cd /home/ec2-user/ansible_on_vyos/ansible_practice/06_condition
```

### 2.仮想環境(poetry)に入る

```shell
$ poetry shell

# Spawning shell within /home/ec2-user/ansible_on_vyos/.venv
```

### 3.インベントリファイルの内容を確認

```ini
[vyos]
vyos01 ansible_host=10.0.0.2
vyos02 ansible_host=10.0.0.3

[vyos:vars]
ansible_network_os=vyos
ansible_connection=network_cli
ansible_user=vyos
ansible_password=vyos

[centos7]
host01 ansible_host=10.0.0.4
host02 ansible_host=10.0.0.5

[centos7:vars]
ansible_user=root
ansible_password=test_password
```

### 4.playbookの内容を確認

- 「when」を使用して、実行対象ノードの条件分岐を実施
- 「when: inventory_hostname == 'vyos01'」とすることで、  
実行対象ノードがvyos01のとき「show ip route」「show interface」を実行する
- showコマンドをdebugさせる際は「when」を使用していない。

```yaml
---
- name: Sample1
  hosts: vyos
  gather_facts: false

  tasks:
    - name: Vyos01 only show commands
      vyos.vyos.vyos_command:
        commands: 
          - show ip route
          - show interface
      register: result
      when: inventory_hostname == 'vyos01'

    - name: Vyos debug show commands
      ansible.builtin.debug: 
        var: result.stdout_lines

```

### 5.playbookを実行

- 実行対象ノード「vyos01」のみ「show ip route」「show interface」を実行
- 実行対象ノード「vyos02」は「show ip route」「show interface」を実行していないので、  
実行結果をdebugしようとするとエラー出力される。

```shell
$ ansible-navigator run when_sample_1.yml -i inventory.ini

PLAY [Sample1] ********************************************************************************************

TASK [Vyos01 only show commands] **************************************************************************
skipping: [vyos02]
ok: [vyos01]

TASK [Vyos debug show commands] ***************************************************************************
ok: [vyos01] => {
    "result.stdout_lines": [
        [
            "Codes: K - kernel route, C - connected, S - static, R - RIP,",
            "       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,",
            "       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,",
            "       F - PBR, f - OpenFabric,",
            "       > - selected route, * - FIB route, q - queued, r - rejected, b - backup",
            "",
            "K>* 0.0.0.0/0 [0/0] via 10.0.0.1, eth0, 00:00:26",
            "C>* 10.0.0.0/24 is directly connected, eth0, 00:00:26",
            "C * 192.168.1.0/24 is directly connected, eth1, 00:00:14",
            "C>* 192.168.1.0/24 is directly connected, eth1, 00:00:26",
            "C * 192.168.2.0/24 is directly connected, eth2, 00:00:14",
            "C>* 192.168.2.0/24 is directly connected, eth2, 00:00:26"
        ],
        [
            "Codes: S - State, L - Link, u - Up, D - Down, A - Admin Down",
            "Interface        IP Address                        S/L  Description",
            "---------        ----------                        ---  -----------",
            "eth0             10.0.0.2/24                       u/u  ",
            "eth1             192.168.1.252/24                  u/u  ",
            "                 192.168.1.254/24                       ",
            "eth2             192.168.2.252/24                  u/u  ",
            "                 192.168.2.254/24                       ",
            "lo               127.0.0.1/8                       u/u"
        ]
    ]
}
ok: [vyos02] => {
    "result.stdout_lines": "VARIABLE IS NOT DEFINED!"
}

PLAY RECAP ************************************************************************************************
vyos01                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vyos02                     : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

$ 
```

<br>
<br>
<br>

---

## condition(条件分岐)についてのまとめ

- 条件分岐をさせるときは、whenディレクティブを使用する。
- 比較演算子、論理演算子、in演算子、is演算子などの条件式を用いて、whenディレクティブが使用されている。

<br>
<br>
<br>

---

## 4.whenディレクティブの演習

---

### Q1 以下の条件でplaybookを作成したとき、空欄に当てはまるものは何でしょう

- 条件
  - vyos01だけに「show configuration」を実行する

- playbook

```yaml
---
- name: Exam1
  hosts: all
  gather_facts: false

  tasks:
    - name: Vyos show configuration
      vyos.vyos.vyos_command:
        commands:
          - show configuration
      when: ■■■
```

<br>
<br>
<br>

---

### Q2 以下のplaybookは、どのような条件のplaybookであるか答えてください

- playbook

```yaml
---
- name: Exam2
  hosts: all

  tasks:
    - name: Exam2 playbook
      ansible.builtin.debug:
        msg: "exam2 playbook test!!"
      when: "'RedHat' in ansible_distribution or 'CentOS' in ansible_distribution"
```

<br>
<br>
<br>

---

### Q3 以下の条件でplaybookを作成して下さい

- 使用インベントリファイル：「/home/ec2-user/ansible_on_vyos/ansible_practice/06_condition」配下のinventory.ini
- playbook作成先ディレクトリ：「/home/ec2-user/ansible_on_vyos/ansible_practice/06_condition」配下
- playbook名：「when_exam_3.yml」で作成
- 実行対象ノード： hosts: all とすること。
- 処理内容：
  - vyos01,vyos02に「show ip route」を実行
  - 実行したとき、host01,host02がskippingされることを確認する

<br>
<br>
<br>

---

### Q4 以下の条件でplaybookを作成して下さい

- 使用インベントリファイル：「/home/ec2-user/ansible_on_vyos/ansible_practice/06_condition」配下のinventory.ini
- playbook作成先ディレクトリ：「/home/ec2-user/ansible_on_vyos/ansible_practice/06_condition」配下
- playbook名：「when_exam_4.yml」で作成
- 実行対象ノード： hosts: all とすること。
- 処理内容：
  - host01の「/tmp」配下に「test_exam4.txt」を作成
  - 「test_exam4.txt」の作成に成功したら、「make success text!」というメッセージを出力させる。

<br>
<br>
<br>

---

### A1. 以下、解答例

- 以下、正しいplaybook
- vyos01だけを実行対象ノードとしたいので、「when: inventory_hostname == vyos01」を指定

```yaml
---
- name: Exam1
  hosts: all
  gather_facts: false

  tasks:
    - name: Vyos show configuration
      vyos.vyos.vyos_command:
        commands:
          - show configuration
      when: inventory_hostname == 'vyos01'
```

<br>
<br>
<br>

---

### A2. 正解：「実行対象ノードのOSがRedHatかCentOSだった場合、「exam2 playbook test!!」というメッセージを出力させる。」

- ansible_distribution（ファクト変数） には、実行対象ノードのOS情報が格納されている

<br>
<br>
<br>

---

### A3.以下、解答例

- playbook

```yaml
---
- name: Exam3
  hosts: all
  gather_facts: false

  tasks:
    - name: Vyos show ip route
      vyos.vyos.vyos_command:
        commands:
          - show ip route
      when: inventory_hostname in ['vyos01','vyos02']
```

- playbookの実行結果

```shell
$ ansible-navigator run answer/when_exam_3.yml -i inventory.ini

PLAY [Exam3] **********************************************************************************************

TASK [Vyos show ip route] *********************************************************************************
skipping: [host01]
skipping: [host02]
ok: [vyos02]
ok: [vyos01]

PLAY RECAP ************************************************************************************************
host01                     : ok=0    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
host02                     : ok=0    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
vyos01                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vyos02                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

$ 
```

<br>
<br>
<br>

---

### A4. 以下、解答例

- playbook

```yaml
---
- name: Exam4
  hosts: all
  gather_facts: false

  tasks:
    - name: Make text file
      ansible.builtin.file:
        path: /tmp/test_exam4.txt
        state: touch
        mode: '0644'
      register: result
      when: inventory_hostname == 'host01'

    - name: Debug success msg
      ansible.builtin.debug:
        msg: "make success text!"
      when: inventory_hostname == 'host01' and result is succeeded
```

- playbookの実行結果

```shell
$ ansible-navigator run answer/when_exam_4.yml -i inventory.ini

PLAY [Exam4] **********************************************************************************************

TASK [Make text file] *************************************************************************************
skipping: [vyos01]
skipping: [vyos02]
skipping: [host02]
changed: [host01]

TASK [Debug success msg] **********************************************************************************
skipping: [vyos01]
skipping: [vyos02]
ok: [host01] => {
    "msg": "make success text!"
}
skipping: [host02]

PLAY RECAP ************************************************************************************************
host01                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
host02                     : ok=0    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
vyos01                     : ok=0    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
vyos02                     : ok=0    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

$ 
```
