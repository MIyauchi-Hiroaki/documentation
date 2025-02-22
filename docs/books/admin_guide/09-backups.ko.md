---
title: 백업 및 복원
---

# 백업 및 복원

이 장에서는 Linux로 데이터를 백업하고 복원하는 방법을 배웁니다.

****

**목표**: 이 문서에서는 미래의 Linux 관리자가 다음을 수행하는 방법을 배웁니다:

:heavy_check_mark: `tar` 및 `cpio` 명령을 사용하여 백업을 만듭니다.   
:heavy_check_mark: 백업을 확인하고 데이터를 복원합니다.   
:heavy_check_mark: 백업을 압축하거나 압축 해제합니다.

:checkered_flag: **백업**, **복원**, **압축**

**지식**: :star: :star: :star:   
**복잡성**: :star: :star:

**소요 시간**: 40분

****

!!! 참고 사항

    이 장 전체에서 명령 구조는 "장치"를 사용하여 백업 대상 위치와 복원 시 소스 위치를 모두 지정합니다. 장치는 외부 미디어 또는 로컬 파일일 수 있습니다. 챕터가 진행됨에 따라 이에 대한 느낌을 얻을 수 있지만 필요한 경우 설명을 위해 언제든지 이 노트를 다시 참조할 수 있습니다.

백업은 확실하고 효과적인 방법으로 데이터를 보존하고 복원해야 하는 필요성에 답할 것입니다.

백업을 통해 다음으로부터 자신을 보호할 수 있습니다.

* **파기**: 자발적 또는 비자발적. 인간 또는 기술. 바이러스, ...
* **삭제**: 자발적 또는 비자발적. 인간 또는 기술. 바이러스, ...
* **무결성**: 데이터를 사용할 수 없게 됩니다.

오류가 없는 시스템은 없으며 사람도 오류가 없으므로 데이터 손실을 방지하려면 문제가 발생한 후 복원할 수 있도록 백업해야 합니다.

재해로 인해 서버와 백업이 파괴되지 않도록 백업 미디어는 서버가 아닌 다른 방(또는 건물)에 보관해야 합니다.

또한 관리자는 미디어를 계속 읽을 수 있는지 정기적으로 확인해야 합니다.

## 개요

**백업**과 **아카이브**의 두 가지 원칙이 있습니다.

* 아카이브는 작업 후 정보 소스를 파기합니다.
* 백업은 작업 후 정보 소스를 보존합니다.

이러한 작업은 파일, 주변 장치 또는 지원되는 미디어(테이프, 디스크 등)에 정보를 저장하는 작업으로 구성됩니다.

### 프로세스

백업에는 시스템 관리자의 많은 규율과 엄격함이 필요합니다. 다음과 같은 질문을 할 필요가 있습니다.

* 적절한 매체는 무엇입니까?
* 무엇을 백업해야 합니까?
* 카피수는 몇개입니까?
* 백업 시간은 얼마나 걸립니까?
* 방법?
* 얼마나 자주?
* 자동 또는 수동?
* 어디에 저장하나요?
* 얼마나 오래 보관됩니까?

### 백업 방법

* **완료**: 하나 이상의 **파일 시스템**이 백업됩니다(커널, 데이터, 유틸리티 등).
* **부분**: 하나 이상의 **파일**이 백업됩니다(구성, 디렉토리 등).
* **Differential**: 마지막 **전체** 백업 이후 수정된 파일만 백업됩니다.
* **Incremental**: 마지막 백업 이후 수정된 파일만 백업됩니다.

### 주기

* **사전에 백업**: 지정된 시간에(시스템 업데이트 전, ...).
* **주기적 백업**: 매일, 매주, 매월, ...

!!! !!!

    시스템을 변경하기 전에 백업해 두는 것이 유용할 수 있습니다. 그러나 매월 변경되는 데이터를 매일 백업하는 것은 의미가 없습니다.

### 복원 방법

사용 가능한 유틸리티에 따라 여러 유형의 복원을 수행할 수 있습니다.

* **완전한 복원**: 트리, ...
* **선택적 복원**: 트리의 일부, 파일, ...

전체 백업을 복원하는 것도 가능하지만 일부만 복원하는 것도 가능합니다. 단, 디렉터리 복원 시 백업 후 생성된 파일은 삭제되지 않습니다.

!!! !!!

    백업 당시의 디렉터리를 복구하려면 복원을 시작하기 전에 해당 내용을 완전히 삭제해야 합니다.

### 도구

백업을 만드는 많은 유틸리티가 있습니다.

* **편집기 도구**;
* **그래픽 도구**;
* **command line 도구**: `tar`, `cpio`, `pax`, `dd`, `dump`, ...

여기서 사용할 명령은 `tar` 및 `cpio`입니다.

* `tar`:
  * 사용하기 쉽습니다.
  * 기존 백업에 파일을 추가할 수 있습니다.
* `cpio`:
  * 소유자를 유지합니다.
  * 그룹, 날짜 및 권리를 유지합니다.
  * 손상된 파일을 건너뜁니다.
  * 완전한 파일 시스템입니다.

!!! 참고 사항

    이러한 명령은 독점적이고 표준화된 형식으로 저장됩니다.

### 명명 규칙

명명 규칙을 사용하면 백업 파일의 내용을 빠르게 대상으로 지정하여 위험한 복원을 방지할 수 있습니다.

* 디렉토리 이름;
* 사용된 유틸리티;
* 사용된 옵션;
* 날짜

!!! !!!

    백업 이름은 명시적인 이름이어야 합니다.

!!! 참고 사항

    Linux에서는 확장이라는 개념이 존재하지 않습니다. 즉, 여기서 확장 기능을 사용하는 것은 인간 운영자를 위한 것입니다. 예를 들어 시스템 관리자가 `.tar.gz` 또는 `.tgz` 파일 확장자를 본다면 그는 파일을 처리하는 방법을 알고 있는 것입니다.

### 백업 내용

백업에는 일반적으로 다음 요소가 포함됩니다.

* 파일
* 이름
* 소유자
* 크기
* 권한
* 액세스 날짜

!!! 참고 사항

    'inode' 번호가 없습니다.

### 스토리지 모드

두 가지 저장 모드가 있습니다.

* 디스크의 파일
* 장치

## Tape ArchiveR - `tar`

`tar` 명령을 사용하면 여러 연속 미디어에 저장할 수 있습니다(다중 볼륨 옵션).

백업의 전체 또는 일부를 추출할 수 있습니다.

`tar`는 절대모드에서 백업할 정보의 경로가 언급되어 있어도 묵시적으로 상대모드로 백업합니다. 그러나 절대 모드의 백업 및 복원은 가능합니다.

### 복원 지침

올바른 질문은 다음과 같습니다.

* 무엇: 부분적이거나 완전한 것;
* 어디에: 데이터가 복원될 장소;
* 방법: 절대 또는 상대.

!!! 주의

    복원을 하기 전에 실수를 피하기 위해 가장 적절한 방법에 대해 생각하고 결정하는 시간을 갖는 것이 중요합니다.

복원은 일반적으로 신속하게 해결해야 하는 문제가 발생한 후에 수행됩니다. 부실한 복원은 경우에 따라 상황을 악화시킬 수 있습니다.

### `tar`로 백업

UNIX 시스템에서 백업을 생성하기 위한 기본 유틸리티는 `tar` 명령입니다. 이러한 백업은 `bzip2`, `xz`, `lzip`, `lzma`, `lzop`, `gzip`, `compress` 또는 `zstd` 로 압축할 수 있습니다.

`tar`를 사용하면 백업에서 단일 파일 또는 디렉토리를 추출하고 내용을 보거나 무결성을 확인할 수 있습니다.

#### 백업 크기 예측

다음 명령은 가능한 _tar_ 파일의 크기를 킬로바이트 단위로 추정합니다.

```
$ tar cf - /directory/to/backup/ | wc -c
20480
$ tar czf - /directory/to/backup/ | wc -c
508
$ tar cjf - /directory/to/backup/ | wc -c
428
```

!!! 주의

    명령줄에 "-"가 있으면 `zsh`가 방해됩니다. 'bash'로 전환하세요!

#### `tar` 백업의 명명 규칙

다음은 이름에 날짜가 추가된다는 것을 알고 있는 `tar` 백업에 대한 명명 규칙의 예입니다.

| 키       | 파일      | Suffix           | 관찰                            |
| ------- | ------- | ---------------- | ----------------------------- |
| `cvf`   | `home`  | `home.tar`       | 상대 모드의 `/home`, 압축되지 않은 형식    |
| `cvfP`  | `/etc`  | `etc.A.tar`      | 절대 모드의 `/etc`, 압축되지 않은 형식     |
| `cvfz`  | `usr`   | `usr.tar.gz`     | 상대 모드의 `/usr`, _gzip_ 압축된 형식  |
| `cvfj`  | `usr`   | `usr.tar.bz2`    | 상대 모드의 `/usr`, _bzip2_ 압축된 형식 |
| `cvfPz` | `/home` | `home.A.tar.gz`  | 절대 모드의 `home`, _gzip_ 압축된 형식  |
| `cvfPj` | `/home` | `home.A.tar.bz2` | 절대 모드의 `home`, _bzip2_ 압축된 형식 |
| …       |         |                  |                               |

#### 백업 생성

##### 상대 모드에서 백업 생성

`cvf` 키를 사용하여 상대 모드에서 압축되지 않은 백업을 생성합니다.

```
tar c[vf] [device] [file(s)]
```

예시:

```
[root]# tar cvf /backups/home.133.tar /home/
```


| 키   | 설명                     |
| --- | ---------------------- |
| `c` | 백업 생성                  |
| `v` | 처리된 파일의 이름을 표시합니다.     |
| `f` | 백업 이름(중간)을 지정할 수 있습니다. |

!!! !!!

    `tar` 키 앞의 하이픈(`-`)은 필요하지 않습니다!

##### 절대 모드에서 백업 생성

절대 모드에서 비압축 백업을 명시적으로 생성하려면 `cvfP` 키를 사용합니다.

```
$ tar c[vf]P [device] [file(s)]
```

예시:

```
[root]# tar cvfP /backups/home.133.P.tar /home/
```

| 키   | 설명                 |
| --- | ------------------ |
| `P` | 절대 모드에서 백업을 생성합니다. |


!!! 주의

    `P` 키로 백업할 파일의 경로를 **절대모드**로 입력해야 합니다. 두 가지 조건(키 `P` 및 **절대경로 **)이 표시되지 않으면 백업은 상대 모드입니다.

##### `gzip`으로 압축 백업 만들기

`gzip`으로 압축된 백업 만들기는 `cvfz` 키로 수행됩니다.

```
$ tar cvzf backup.tar.gz dirname/
```

| 키   | 설명                 |
| --- | ------------------ |
| `z` | _gzip_에 백업을 압축합니다. |


!!! 참고 사항

    `.tgz` 확장자는 `.tar.gz`와 동등한 확장자입니다.

!!! 참고 사항

    모든 백업 작업에 대해 `cvf`(`tvf` 또는 `xvf`) 키를 변경하지 않고 압축 키를 키 끝에 추가하면 명령을 더 쉽게 이해할 수 있습니다(예: `cvfz` 또는 `cvfj` 등). .).

##### `bzip`으로 압축 백업 만들기

`bzip`을 사용하여 압축된 백업을 생성하려면 `cvfj` 키를 사용합니다.

```
$ tar cvfj backup.tar.bz2 dirname/
```

| 키   | 설명                  |
| --- | ------------------- |
| `j` | _bzip2_에 백업을 압축합니다. |

!!! 참고 사항

    `.tbz` 및 `.tb2` 확장자는 `.tar.bz2` 확장자와 동일합니다.

##### 압축 `compress`, `gzip`, `bzip2`, `lzip` 및 `xz`

압축 및 그에 따른 압축 해제는 리소스 소비(시간 및 CPU 사용량)에 영향을 미칩니다.

다음은 가장 효율적인 것부터 가장 효율적인 것까지 텍스트 파일 집합의 압축 순위입니다.

- compress (`.tar.Z`)
- gzip (`.tar.gz`)
- bzip2 (`.tar.bz2`)
- lzip (`.tar.lz`)
- xz (`.tar.xz`)

#### 기존 백업에 파일 또는 디렉터리 추가

기존 백업에 하나 이상의 항목을 추가할 수 있습니다.

```
tar {r|A}[key(s)] [device] [file(s)]
```

백업 `/backups/home.133.tar`에 `/etc/passwd`를 추가하려면:

```
[root]# tar rvf /backups/home.133.tar /etc/passwd
```

디렉토리 추가도 비슷합니다. 여기서 `dirtoadd`를 `backup_name.tar`에 추가합니다.

```
$ tar rvf backup_name.tar dirtoadd
```

| 키   | 설명                                         |
| --- | ------------------------------------------ |
| `r` | 직접 액세스 미디어 백업(하드 디스크) 끝에 하나 이상의 파일을 추가합니다. |
| `A` | 순차 액세스 미디어(테이프)의 백업 끝에 하나 이상의 파일을 추가합니다.   |

!!! 참고 사항

    압축된 백업에는 파일이나 폴더를 추가할 수 없습니다.

    ```
    $ tar rvfz backup.tgz filetoadd
    tar: Cannot update compressed archives
    Try `tar --help' or `tar --usage' for more information.
    ```

!!! 참고 사항

    백업이 상대 모드에서 수행된 경우 상대 모드에서 파일을 추가합니다. 백업이 절대 모드에서 수행된 경우 절대 모드에서 파일을 추가하십시오.
    
    믹싱 모드는 복원할 때 문제를 일으킬 수 있습니다.

#### 백업 내용 나열

압축을 풀지 않고 백업 내용을 볼 수 있습니다.

```
tar t[key(s)] [device]
```

| 키   | 설명                   |
| --- | -------------------- |
| `t` | 백업 내용(압축 여부)을 표시합니다. |

예시:

```
$ tar tvf backup.tar
$ tar tvfz backup.tar.gz
$ tar tvfj backup.tar.bz2
```

백업의 파일 수가 많아지면 `tar` 명령의 결과를 _호출기_에 _파이프_ 할 수 있습니다(`more`, `less`, `most`, 등.).

```
$ tar tvf backup.tar | less
```

!!! !!!

    백업 내용을 나열하거나 검색하기 위해 백업을 만들 때 사용된 압축 알고리즘을 언급할 필요가 없습니다. 즉, `tar tvf`는 내용을 읽는 `tar tvfj`에 해당하고 `tar xvf`는 추출하기 위한 `tar xvfj`에 해당합니다.

!!! !!!

    항상 백업 내용을 확인하십시오.

#### 백업 무결성 확인

백업의 무결성은 생성 시 `W` 키로 테스트할 수 있습니다.

```
$ tar cvfW file_name.tar dir/
```

백업의 무결성은 생성 후 키 `d`로 테스트할 수 있습니다.

```
$ tar vfd file_name.tar dir/
```

!!! !!!

    이전 키에 두 번째 `v`를 추가하면 보관된 파일 목록과 보관된 파일과 파일 시스템에 있는 파일 간의 차이점을 얻을 수 있습니다.

    ```
    $ tar vvfd  /tmp/quodlibet.tar .quodlibet/
    drwxr-x--- rockstar/rockstar     0 2021-05-21 00:11 .quodlibet/
    -rw-r--r-- rockstar/rockstar     0 2021-05-19 00:59 .quodlibet/queue
    […]
    -rw------- rockstar/rockstar  3323 2021-05-21 00:11 .quodlibet/config
    .quodlibet/config: Mod time differs
    .quodlibet/config: Size differs
    […]
    ```

`W` 키는 아카이브의 내용을 파일 시스템과 비교하는 데에도 사용됩니다.

```
$ tar tvfW file_name.tar
Verify 1/file1
1/file1: Mod time differs
1/file1: Size differs
Verify 1/file2
Verify 1/file3
```

`W` 키를 사용한 확인은 압축된 아카이브로 수행할 수 없습니다. `d` 키를 사용해야 합니다.

```
$ tar dfz file_name.tgz
$ tar dfj file_name.tar.bz2
```

#### 추출(_untar_)하여 백업

백업 추출(_untar_) `*.tar`은 `xvf` 키로 수행됩니다.

`/savings/etc.133.tar` 백업에서 활성 디렉토리의 `etc` 디렉토리로 `etc/exports` 파일을 추출하십시오.

```
$ tar xvf /backups/etc.133.tar etc/exports
```

압축된 백업 `/backups/home.133.tar.bz2`에서 활성 디렉터리로 모든 파일을 추출합니다.

```
[root]# tar xvfj /backups/home.133.tar.bz2
```

백업 `/backups/etc.133.P.tar`에서 원래 디렉터리로 모든 파일을 추출합니다.

```
$ tar xvfP /backups/etc.133.P.tar
```

!!! 주의

    올바른 장소로 이동하십시오.
    
    백업 내용을 확인하십시오.

| 키   | 설명                           |
| --- | ---------------------------- |
| `x` | 압축 여부에 관계없이 백업에서 파일을 추출하십시오. |


_tar-gzipped_(`*.tar.gz`) 백업 추출은 `xvfz` 키를 사용하여 수행됩니다.

```
$ tar xvfz backup.tar.gz
```

_tar-bzipped_(`*.tar.bz2`) 백업 추출은 `xvfj` 키를 사용하여 수행됩니다.

```
$ tar xvfj backup.tar.bz2
```

!!! !!!

    백업 내용을 추출하거나 나열하기 위해 백업 생성에 사용된 압축 알고리즘을 언급할 필요가 없습니다. 즉, 'tar xvf'는 콘텐츠를 추출하기 위한 'tar xvfj'와 동일하고, 'tar tvf'는 목록을 위한 'tar tvfj'와 동일합니다.

!!! 주의

    파일을 원래 디렉토리(`tar xvf`의 `P` 키) 에 복원하려면 절대 경로로 백업을 생성해야 합니다. 즉, `tar cvf`의 `P` 키를 사용합니다.

##### _tar_ 백업에서 파일만 추출

_tar_ 백업에서 특정 파일을 추출하려면 `tar xvf` 명령 끝에 해당 파일의 이름을 지정하십시오.

```
$ tar xvf backup.tar /path/to/file
```

이전 명령은 `backup.tar` 백업에서 `/path/to/file` 파일만 추출합니다. 이 파일은 활성 디렉토리에 생성되었거나 이미 존재하는 `/path/to/` 디렉토리로 복원됩니다.

```
$ tar xvfz backup.tar.gz /path/to/file
$ tar xvfj backup.tar.bz2 /path/to/file
```

##### 백업 _tar_에서 폴더 추출

백업에서 하나의 디렉토리(하위 디렉토리 및 파일 포함)만 추출하려면 `tar xvf` 명령 끝에 디렉토리 이름을 지정하십시오.

```
$ tar xvf backup.tar /path/to/dir/
```

여러 디렉터리를 추출하려면 각 이름을 차례로 지정합니다.

```
$ tar xvf backup.tar /path/to/dir1/ /path/to/dir2/
$ tar xvfz backup.tar.gz /path/to/dir1/ /path/to/dir2/
$ tar xvfj backup.tar.bz2 /path/to/dir1/ /path/to/dir2/
```

##### 정규식(_regex_) 을 사용하여 _tar_ 백업에서 파일 그룹 추출

_regex_를 지정하여 지정된 선택 패턴과 일치하는 파일을 추출합니다.

예를 들어 확장자가 `.conf`인 모든 파일을 추출하려면 다음과 같이 하십시오.

```
$ tar xvf backup.tar --wildcards '*.conf'
```

키:

  * **--wildcards *.conf**는 확장자가 `.conf`인 파일에 해당합니다.

## _CoPy Input Output_ - `cpio`

`cpio` 명령을 사용하면 옵션을 지정하지 않고 여러 연속 미디어에 저장할 수 있습니다.

백업의 전체 또는 일부를 추출할 수 있습니다.

`tar` 명령과 달리 백업과 압축을 동시에 할 수 있는 옵션은 없습니다. 따라서 백업과 압축의 두 단계로 이루어집니다.

`cpio`를 사용하여 백업을 수행하려면 백업할 파일 목록을 지정해야 합니다.

이 목록은 `find`, `ls` 또는 `cat` 명령과 함께 제공됩니다.

* `find` : 재귀적이든 아니든 트리 탐색;
* `ls` : 재귀적이든 아니든 디렉터리를 나열;
* `cat` : 저장할 트리 또는 파일이 포함된 파일을 읽기.

!!! 참고 사항

    `ls`는 `-l`(세부 정보) 또는 `-R`(재귀)과 함께 사용할 수 없습니다.
    
    간단한 이름 목록이 필요합니다.

### `cpio` 명령으로 백업 생성

`cpio` 명령 구문:

```
[files command || cpio {-o| --create} [-options] [<file-list] [>device]
```

예시:

`cpio`의 출력 리디렉션:

```
$ find /etc | cpio -ov > /backups/etc.cpio
```

백업 미디어 이름 사용:

```
$ find /etc | cpio -ovF /backups/etc.cpio
```

`find` 명령의 결과는 _pipe_를 통해 `cpio` 명령에 대한 입력으로 전송됩니다(문자`|`, <kbd>AltGr</kbd> + <kbd>6</kbd>).

여기서 `find /etc` 명령은 `/etc` 디렉터리의 내용에 해당하는 파일 목록을 백업을 수행하는 `cpio` 명령으로 (재귀적으로) 반환합니다.

저장할 때 `>` 기호나 `F save_name_cpio`를 잊지 마십시오.

| 옵션   | 설명                  |
| ---- | ------------------- |
| `-o` | 백업(_출력_)을 생성합니다.    |
| `-v` | 처리된 파일의 이름을 표시합니다.  |
| `-F` | 변경할 백업을 지정합니다(미디어). |

미디어에 백업:

```
$ find /etc | cpio -ov > /dev/rmt0
```

지원에는 여러 가지 유형이 있습니다.

* tape drive: `/dev/rmt0`;
* a partition: `/dev/sda5`, `/dev/hda5`, etc.

### 백업 유형

#### 상대 경로로 백업

```
$ cd /
$ find etc | cpio -o > /backups/etc.cpio
```

#### 절대 경로로 백업

```
$ find /etc | cpio -o > /backups/etc.A.cpio
```

!!! 주의

    `find` 명령에 지정된 경로가 **absolute**인 경우 **absolute**에서 백업이 수행됩니다.
    
    `find` 명령에 표시된 경로가 **relative**인 경우 **relative**로 백업이 수행됩니다.

### 백업에 추가

```
[files command |] cpio {-o| --create} -A [-options] [<fic-list] {F|>device}
```

예시:

```
$ find /etc/shadow | cpio -o -AF SystemFiles.A.cpio
```

파일 추가는 직접 액세스 미디어에서만 가능합니다.

| 옵션   | 설명                         |
| ---- | -------------------------- |
| `-A` | 디스크의 백업에 하나 이상의 파일을 추가합니다. |
| `-F` | 수정할 백업을 지정합니다.             |

### 백업 압축

* 저장 **후** 압축

```
$ find /etc | cpio  –o > etc.A.cpio
$ gzip /backups/etc.A.cpio
$ ls /backups/etc.A.cpio*
/backups/etc.A.cpio.gz
```

* 저장 **및** 압축

```
$ find /etc | cpio –o | gzip > /backups/etc.A.cpio.gz
```

`tar` 명령과 달리 저장과 압축을 동시에 할 수 있는 옵션은 없습니다. 따라서 저장과 압축의 두 단계로 이루어집니다.

첫 번째 방법의 구문은 두 단계로 수행되므로 이해하고 기억하기가 더 쉽습니다.

첫 번째 방법의 경우 파일 이름 끝에 `.gz`를 추가하는 `gzip` 유틸리티에 의해 백업 파일 이름이 자동으로 변경됩니다. 마찬가지로 `bzip2` 유틸리티는 `.bz2`를 자동으로 추가합니다.

### 백업 내용 읽기

_cpio_ 백업의 내용을 읽기 위한 `cpio` 명령의 구문:

```
cpio -t [-options] [<fic-list]
```

예시:

```
$ cpio -tv </backups/etc.152.cpio | less
```

| 옵션   | 설명            |
| ---- | ------------- |
| `-t` | 백업을 읽습니다.     |
| `-v` | 파일 속성을 표시합니다. |

백업을 만든 후에는 내용을 읽어 오류가 없는지 확인해야 합니다.

마찬가지로 복원을 수행하기 전에 사용할 백업의 내용을 읽어야 합니다.

### 백업 복원

백업을 복원하기 위한 `cpio` 명령의 구문:

```
cpio {-i| --extract} [-E file] [-options] [<device]
```

예시:

```
$ cpio -iv </backups/etc.152.cpio | less
```

| 옵션                           | 설명                                   |
| ---------------------------- | ------------------------------------ |
| `-i`                         | 전체 백업을 복원합니다.                        |
| `-E file`                    | 파일에 이름이 포함된 파일만 복원합니다.               |
| `--make-directories` 또는 `-d` | 누락된 트리 구조를 재구축합니다.                   |
| `-u`                         | 존재하는 경우에도 모든 파일을 바꿉니다.               |
| `--no-absolute-filenames`    | 절대 모드에서 만든 백업을 상대적인 방식으로 복원할 수 있습니다. |

!!! 주의

    기본적으로 복원 시 마지막 수정 날짜가 백업 날짜보다 최근이거나 같은 Disk의 파일은 복원되지 않습니다(이전 정보로 최근 정보를 덮어쓰는 것을 방지하기 위해).
    
    반면에 `u` 옵션을 사용하면 파일의 이전 버전을 복원할 수 있습니다.

예시:

* 절대 백업의 절대 복원

```
$ cpio –ivF home.A.cpio
```

* 기존 트리 구조에 절대 복원

`u` 옵션을 사용하면 복원이 수행되는 위치에서 기존 파일을 덮어쓸 수 있습니다.

```
$ cpio –iuvF home.A.cpio
```

* 상대 모드에서 절대 백업 복원

긴 옵션 `no-absolute-filenames`은 상대 모드에서 복원을 허용합니다. 실제로 경로 시작 부분의 `/`가 제거됩니다.

```
$ cpio --no-absolute-filenames -divuF home.A.cpio
```

!!! 팁

    디렉토리 생성이 필요할 수 있으므로 `d` 옵션을 사용합니다.

* 상대 백업 복원

```
$ cpio –iv <etc.cpio
```

* 파일 또는 디렉토리의 절대 복원

특정 파일이나 디렉터리를 복원하려면 삭제해야 하는 목록 파일을 만들어야 합니다.

```
echo "/etc/passwd" > tmp
cpio –iuE tmp -F etc.A.cpio
rm -f tmp
```

## 압축 - 압축 해제 유틸리티

백업 시 압축을 사용하면 여러 가지 단점이 있을 수 있습니다.

* 백업 시간과 복원 시간이 늘어납니다.
* 백업에 파일을 추가할 수 없습니다.

!!! 참고 사항

    따라서 백업 중에 압축하는 것보다 백업을 만들어 압축하는 것이 좋습니다.

### `gzip`으로 압축하기

`gzip` 명령은 데이터를 압축합니다.

`gzip` 명령 구문:

```
gzip [options] [file ...]
```

예시:

```
$ gzip usr.tar
$ ls
usr.tar.gz
```

파일은 `.gz` 확장자를 받습니다.

동일한 권한과 동일한 마지막 액세스 및 수정 날짜를 유지합니다.

### `bunzip2`로 압축하기

`bunzip2` 명령도 데이터를 압축합니다.

`bzip2` 명령 구문:

```
bzip2 [options] [file ...]
```

예시:

```
$ bzip2 usr.cpio
$ ls
usr.cpio.bz2
```

파일 이름에는 `.bz2` 확장자가 지정됩니다.

`bzip2`로 압축하는 것이 `gzip`으로 압축하는 것보다 낫지만 실행하는 데 시간이 더 걸립니다.

### `gunzip`으로 압축 해제

`gunzip` 명령은 압축된 데이터의 압축을 풉니다.

`gunzip` 명령 구문:

```
gunzip [options] [file ...]
```

예시:

```
$ gunzip usr.tar.gz
$ ls
usr.tar
```

파일 이름은 `gunzip`에 의해 잘리고 `.gz` 확장자는 제거됩니다.

`gunzip`은 또한 다음 확장자를 가진 파일의 압축을 풉니다.

* `.z`;
* `-z`;
* `_z` .

### `bunzip2`로 압축 해제

`bunzip2` 명령은 압축된 데이터의 압축을 풉니다.

`bzip2` 명령 구문:

```
bzip2 [options] [file ...]
```

예시:

```
$ bunzip2 usr.cpio.bz2
$ ls
usr.cpio
```

파일 이름은 `bunzip2`에 의해 잘리고 확장자 `.bz2`는 제거됩니다.

`bunzip2`는 또한 다음 확장자를 가진 파일의 압축을 풉니다.

* `-bz` ;
* `.tbz2` ;
* `tbz` .
