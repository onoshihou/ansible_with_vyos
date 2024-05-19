
# day5 variable

- variable(変数)を学習する

- 目次
  - 1.variable(変数)とは？
  - 2.variable(変数)の説明
  - 3.variable(変数)の実習[ハンズオン]
  - variable(変数)のまとめ
  - 4.variable(変数)の演習問題

<br>
<br>
<br>

---

## 1. variable(変数)とは？

- 特定の文字列に対して様々な値を入れることができる箱のようなもの。<br>
→このように変数に対して文字列や数字などの値を入れることを**変数定義**という。

- 数学でよく見る以下の式も変数の一種である。

```python
x = 2      <-「x」という変数は数字の「2」として定義されている

y = 3      <-「y」という変数は数字の「3」として定義されている

x + y = 5  <-「x=2」+「y=3」のため右辺は「5」となる
```

- ただし、プログラミング上での変数定義では必ずしも左辺と右辺がイコールになるわけではない。
- 以下に、プログラミングならではの変数定義例を記載する。

```python
$ python3

>>> x = 2       <-A. 「x」という変数は数字の「2」として定義されている
>>> x = 3 + x   <-B. 「x」という変数は新たに「3 + x」として定義される
>>> print(x)　　<-C. 　BではA時点で定義された「x=2」が右辺の「x」に代入されるため、Cの「x」の出力結果は「3 + 2 = 5」となる
5
>>> 
```

<br>
<br>
<br>

---

## 2.variable(変数)の説明

- playbookにおいて任意の値を定義する方法は以下の2つである。

| 変数名 | 説明 |
| :----- | :---------------------- |
| `vars` | Varsセクションで変数定義する。 |
| `set_fact` | Tasksセクションで変数定義する。 |

### 「vars」の使用例

- varは「variable(変数)」の略で、`vars`とは変数を集めておく場所のようなもの。
- 以下にplaybook内のVarsセクションにてよく使用される`vars`について紹介する。
- 下記のパラメータにより定義された変数はTasksセクションで使用することができる。

- varsの使用例を以下に記載。

```yaml
---
- name: Sample_vars
  hosts: vyos01
  gather_facts: false

  vars:
    sample: "Hello!!"

  tasks:
    - name: Test debug vars
      ansible.builtin.debug:
        var: sample
```

### 「set_factモジュール」の使用例

- Tasksセクションで変数定義をしたい時に使用する。

- `set_fact`の使用例を以下に記載。

```yaml
---
- name: Sample_set_fact
  hosts: vyos01
  gather_facts: false

  tasks:
    - name: Test setting variable set_fact
      ansible.builtin.set_fact:
        test: "Hello!!"
```

### Ansibleが用意している、デフォルトで定義されている変数

- 変数は任意の値を定義することが多いが、デフォルトで定義されているものも存在する。

| 変数の種類 | 説明 |
| :----- | :---------------------- |
| マジック変数  | ユーザが直接設定することのできない変数。<br>Ansibleがシステム内の状態を反映してこの変数を常に上書きしている。 |
| ファクト変数  | 現在実行中のホストに関連する情報を含む変数。<br> |
| 接続変数 | ターゲットホストへのアクション実行方法を具体的に設定する時に使用。 |

<br>

変数についてのAnsibleの公式ドキュメントは[こちら](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)

### マジック変数

- 以下にマジック変数の中でも特に使用頻度が高いものを紹介する。

| 変数名 | 説明 |
| :----- | :---------------------- |
| `inventory_hostname` | 現在実行中のホストのイベントリー名。 |
| `ansible_play_name` | 現在実行されているplaybookのplayの名前。 |
| `playbook_dir` | 現在実行されているplaybookのディレクトリパス。 |
| `inventory_dir` | インベントリファイルのディレクトリパス。 |

### ファクト変数

- 以下にファクト変数の一部を紹介する。

| 変数名 | 説明 |
| :----- | :---------------------- |
| `ansible_facts` | `gather_facts: false` の場合はターゲットホストのファクト情報が格納されない。<br>`gather_facts: false` であっても、未定義ではなく、値は空になる。 |

### 接続変数

- 以下に接続変数の中でも特に使用頻度が高いものを紹介する。

<br>

| 変数名 | 説明 |
| :----- | :---------------------- |
| `ansible_network_os` | ターゲットホストのプラットフォーム情報。 |
| `ansible_connection` | ターゲットホストへの接続方式。 |
| `ansible_user` | SSH接続するときのユーザ情報。 |
| `ansible_password` | SSH接続するときのパスワード情報。 |
| `become` | root権限昇格の有無。 |

```ini
[vyos]
vyos01 ansible_host=10.0.0.2
vyos02 ansible_host=10.0.0.3

[vyos:vars]                       # <-vyosグループで指定されたホストへの接続で使用するvarsを定義
ansible_network_os=vyos
ansible_connection=network_cli
ansible_user=vyos
ansible_password=vyos
```

---

### register

- 実行タスクの結果を格納する
- 主な使用方法は、実行結果を`register`で変数に詰めて、その変数の内容から処理を変えたりエラーメッセージを出力するなど多岐にわたる。
- 以下はplaybook作成例である。

```yaml
---
- name: Sample_register
  hosts: vyos01
  gather_facts: false

  tasks:
    - name: Get show version
      vyos.vyos.vyos_command:
        commands: 
          - show version
      register: result

    - name: Debug result
      ansible.builtin.debug:
        var: result.stdout_lines[0]         
```

- 前ページのplaybook実行例を以下に記載。

```shell
PLAY [Sample_register] ************************************************************************************************************************************

TASK [Get show version] ***********************************************************************************************************************************
ok: [vyos01]

TASK [Debug result] ***************************************************************************************************************************************
ok: [vyos01] => {
    "result.stdout_lines[0]": [
        "Version:          VyOS 1.3-rolling-202303060135",
        "Release train:    equuleus",
        "",
        "Built by:         vyos_bld@bb8a87a5ede0",
        "Built on:         Mon 06 Mar 2023 01:35 UTC",
        "Build UUID:       7bb0f480-2516-4689-b0a3-12967c6c54a9",
        "Build commit ID:  3ffe9a2689632b",
        "",
        "Architecture:     x86_64",
        "Boot via:         installed image",
        "System type:      Xen HVM guest",
        "",
        "Hardware vendor:  Xen",
        "Hardware model:   HVM domU",
        "Hardware S/N:     ec246115-cfc8-cb39-f53e-9a00ef3fc5d7",
        "Hardware UUID:    ec246115-cfc8-cb39-f53e-9a00ef3fc5d7",
        "",
        "Copyright:        VyOS maintainers and contributors"
    ]
}

PLAY RECAP ************************************************************************************************************************************************
vyos01                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

<br>
<br>
<br>

---

## 3. variable(変数)の実習

### 目的

- 1. Varsセクションにて`vars`を使用して変数定義を行い、変数の中身をdebugで出力する
- 2. `set_fact`を使用して変数定義を行い、変数の中身をdebugで出力する
- 3. マジック変数の中身をdebugで出力する
- 4. ファクト変数の中身、また一部をdebugで出力する

### 1.ディレクトリ移動

- 使用するplaybook,inventoryファイルが存在するディレクトリに移動

```shell
cd /home/ec2-user/ansible_on_vyos/ansible_practice/05_variable
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
```

### 目的1 Varsセクションにて`vars`を使用して変数定義を行い、変数の中身をdebugで出力する

### 4.playbookの内容を確認

- 「test」という変数に「Hello Ansible!」という文字列を定義
- Varsセクションで定義した「test」の変数の中身を出力

```yaml
---
- name: Sample1_vars
  hosts: localhost
  gather_facts: false

  vars:
    test: "Hello Ansible!"

  tasks:
    - name: Setting vars debug
      ansible.builtin.debug:
        var: test
```

### 5.playbookを実行

```shell
$ ansible-navigator run variable_sample_1.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Sample1_vars] ***************************************************************************************************************************************

TASK [Setting vars debug] *********************************************************************************************************************************
ok: [localhost] => {
    "test": "Hello Ansible!"
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
$ 
```

### 目的2 set_factを使用して変数定義を行い、変数の中身をdebugで出力する

### 4.playbookの内容を確認

- 「set_fact」を使用して「test」という変数に「Hello Ansible!」という文字列を定義
- 「set_fact」で定義した「test」の変数の中身を出力

```yaml
---
- name: Sample2_set_fact
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Setting set_fact
      ansible.builtin.set_fact:
        test: "Hello Ansible!"

    - name: Setting set_fact debug
      ansible.builtin.debug:
        var: test
```

### 5.playbookを実行

```shell
$ ansible-navigator run variable_sample_2.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Sample2_set_fact] ***********************************************************************************************************************************

TASK [Setting set_fact] ***********************************************************************************************************************************
ok: [localhost]

TASK [Setting set_fact debug] *****************************************************************************************************************************
ok: [localhost] => {
    "test": "Hello Ansible!"
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
$ 
```

### 目的3 マジック変数の中身をdebugで出力する

### 4.playbookの内容を確認

- 「ansible_play_name」はマジック変数のため、変数定義の必要なし

```yaml
---
- name: Sample3_magic_variable
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Magic_variable debug
      ansible.builtin.debug:
        var: ansible_play_name
```

### 5.playbookを実行

- 「ansible_play_name」に格納されたplaybookの名前(nameで定義した内容)が出力されていることを確認

```shell
$ ansible-navigator run variable_sample_3.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Sample3_magic_variable] *****************************************************************************************************************************

TASK [Magic_variable debug] *******************************************************************************************************************************
ok: [localhost] => {
    "ansible_play_name": "Sample3_magic_variable"
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
$ 
```

### 目的4 ファクト変数の中身の一部をdebugで出力する

### 4.playbookの内容を確認

- 「gather_facts: false」を省略した場合は「ansible_facts」にファクト情報が格納される。

```yaml
---
- name: Sample4_ansible_facs
  hosts: vyos01

  tasks:
    - name: Debug ansible_facts
      ansible.builtin.debug:
        var: ansible_facts

    - name: Debug ansible_facts.net_hostname
      ansible.builtin.debug:
        var: ansible_facts.net_hostname
```

### 5.playbookを実行

- 「ansible_facts」に格納されているディクショナリの中の値(value)を取り出すことができる

```shell
$ ansible-navigator run variable_sample_4.yml -i inventory.ini

PLAY [Sample4_ansible_facs] *******************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************
ok: [vyos01]

TASK [Debug ansible_facts] ********************************************************************************************************************************
ok: [vyos01] => {
    "ansible_facts": {
        "net_api": "cliconf",
        "net_gather_network_resources": [],
        "net_gather_subset": [
            "default"
        ],
        "net_hostname": "vyos",
        "net_model": "HVM",
        "net_python_version": "3.9.16",
        "net_serialnum": "ec246115-cfc8-cb39-f53e-9a00ef3fc5d7",
        "net_system": "vyos",
        "net_version": "VyOS 1.3-rolling-202303060135",
        "network_resources": {}
    }
}

TASK [Debug ansible_facts.net_hostname] *******************************************************************************************************************
ok: [vyos01] => {
    "ansible_facts.net_hostname": "vyos"
}

PLAY RECAP ************************************************************************************************************************************************
vyos01                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
$ 
```

<br>
<br>
<br>

---

## variable(変数)についてのまとめ

- playbookにおいて任意の値を変数定義する方法は主に2通りある
  - `vars`で定義
  - `set_fact`で定義

- playbookにおいてデフォルトで定義されているものとして以下が存在する
  - マジック変数
  - ファクト変数
  - 接続変数

<br>
<br>
<br>

---

## 4.variable(変数)の演習

---

### Q1 以下のplaybookを実行し、出力された実行結果の空欄に当てはまるものは何でしょう

- playbook

```yaml
---
- name: Exam1
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Setting set_fact
      ansible.builtin.set_fact:
        HelIo: "Hello Ansible!"

    - name: Debug set_fact
      ansible.builtin.debug:
        var: Hello
```

- 実行結果

```shell
$ ansible-navigator run 05_variable/variable_exam_1.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Exam1] **********************************************************************************************************************************************

TASK [Setting set_fact] ***********************************************************************************************************************************
ok: [localhost]

TASK [Debug set_fact] *************************************************************************************************************************************
ok: [localhost] => {
    "Hello": ■■■■
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
$
```

1. “VARIABLE IS NOT DEFINED!”
2. "Hello"
3. "Hello Ansible!"
4. "error"

<br>
<br>
<br>

---

### Q2 以下のplaybookを実行し、出力された実行結果の空欄に当てはまるものは何でしょう

- playbook

```yaml
---
- name: Exam2
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Setting set_fact
      ansible.builtin.set_fact:
        ansible_play_name: "Hello Ansible!"

    - name: Debug variable
      ansible.builtin.debug:
        var: ansible_play_name
```

- 実行結果

```shell
$ ansible-navigator run 05_variable/variable_exam_2.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Exam2] **********************************************************************************************************************************************

TASK [Setting set_fact] ***********************************************************************************************************************************
ok: [localhost]

TASK [Debug variable] *************************************************************************************************************************************
ok: [localhost] => {
    "ansible_play_name": ■■■■
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
$ 
```

1. Hello Ansible!
2. "Hello Ansible!"
3. "variable_exam_2"
4. "Exam2"

<br>
<br>
<br>

---

### Q3 以下の条件のplaybookを作成して、実行してください

- 使用インベントリファイル：「/home/ec2-user/ansible_on_vyos/ansible_practice/05_variable」配下のinventory.ini
- playbook作成先ディレクトリ：「/home/ec2-user/ansible_on_vyos/ansible_practice/05_variable」配下
- playbook名：「variable_exam_3.yml」で作成
- 実行対象ノード：処理内容を満たしていれば何でも可
- 処理内容：
  - debugモジュールの`var`パラメータを使用し、playbook実行時に`TASK [debug]`にて「vyos01」と表示させる

<br>
<br>
<br>

---

### A1 正解：「“VARIABLE IS NOT DEFINED!”」

- 以下、正しい実行結果

```shell
$ ansible-navigator run variable_exam_1.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Exam1] **********************************************************************************************************************************************

TASK [Setting set_fact] ***********************************************************************************************************************************
ok: [localhost]

TASK [Debug set_fact] *************************************************************************************************************************************
ok: [localhost] => {
    "Hello": "VARIABLE IS NOT DEFINED!"
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
$ 
```

- 解説
  - よく変数部分をみてみると`set_fact`で定義している変数名が「Hello」ではなく、「HelIo」になっている。
  - 変数名が間違えていたため、「Hello」は定義されない。
  - 定義されていない変数を出力しようとすると、(VARIABLE IS NOT DEFINED!)と表示される。

<br>
<br>
<br>

---

### A2 正解：「4.Exam2」

- 以下、正しい実行結果

```shell
$ ansible-navigator run variable_exam_2.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Exam2] **********************************************************************************************************************************************

TASK [Setting set_fact] ***********************************************************************************************************************************
ok: [localhost]

TASK [Debug variable] *************************************************************************************************************************************
ok: [localhost] => {
    "ansible_play_name": "Exam2"
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
$ 
```

- `set_fact`で定義した`ansible_play_name`という変数はマジック変数であり、現在実行されているplaybookの名前を変数の中に格納する。
- 今回`set_fact`で「Hello Ansible!」という文字列を定義したが、それよりもマジック変数が優先されるため、出力結果は`Exam2`となる。

<br>
<br>
<br>

---

### A3 以下、解答例

- playbook

```yaml
---
- name: Exam3
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Setting set_fact
      ansible.builtin.set_fact:
        test_hostname: "vyos01"

    - name: Debug test_hostname
      ansible.builtin.debug:
        var: test_hostname
```

- playbookの実行結果

```shell
$ ansible-navigator run variable_exam_3.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Exam3] **********************************************************************************************************************************************

TASK [Setting set_fact] ***********************************************************************************************************************************
ok: [localhost]

TASK [Debug test_hostname] ********************************************************************************************************************************
ok: [localhost] => {
    "test_hostname": "vyos01"
}

PLAY RECAP ************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
$ 
```
