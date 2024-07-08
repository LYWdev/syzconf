# syzconf

### 사용법

1. 환경에 맞게 raw파일 수정
test.sh 파일을 열어 보면 
    ```
            \"workdir\": \"/home/test2004/git_work/syzkaller/workdir\",
            \"syzkaller\": \"/home/test2004/git_work/syzkaller\",
    ```
    위 부분을 각자의 환경에 맞게 파일을 수정하여야함.
    아래는 test.sh
    ```
    #!/bin/bash
    # 시스템 콜 목록 파일 경로
    syscalls_file="syslist_raw.csv"
    # 시스템 콜 배열
    syscalls=()
    # 파일에서 시스템 콜 추출
    while IFS= read -r line; do
        # 첫 번째 행은 시스템 콜 이름이 아닌 'sysname' 라벨이므로 건너뛴다
        if [[ $line == "sysname" ]]; then
            continue
        fi
        # 시스템 콜 이름 추출
        syscall_name="$line"
        # 시스템 콜 배열에 추가
        syscalls+=("$syscall_name")
    done < "$syscalls_file"
    # sysraw 파일 읽기
    sysraw_data=$(cat sysraw)
    # 각 시스템 콜별 설정 파일 생성 및 편집
    for syscall in "${syscalls[@]}"; do
        # 설정 파일 이름 생성
        config_file="sys_$syscall.json"
        # 기본 설정 JSON 문자열 생성
        config_json="{
            \"target\": \"linux/amd64\",
            \"http\": \"127.0.0.1:56741\",
            \"rpc\": \"127.0.0.1:0\",
            \"workdir\": \"/home/test2004/git_work/syzkaller/workdir\",
            \"syzkaller\": \"/home/test2004/git_work/syzkaller\",
            \"enable_syscalls\": [\"$syscall\"],
            \"sandbox\": \"none\",
            \"type\": \"none\",
            \"cover\": false
        }"
        # 기존 sysraw 파일 내용에서 해당 시스템 콜 설정 추출 (있으면)
        syscall_config=$(echo "$sysraw_data" | jq ".enable_syscalls[] | select(\"$syscall\" == .)")
        # 기존 설정이 있으면 반영
        if [[ ! -z "$syscall_config" ]]; then
            config_json=$(echo "$config_json" | jq ".enable_syscalls = [$syscall_config]")
        fi
        # 설정 파일 생성 및 내용 쓰기
        echo "$config_json" > "$config_file"
    done

    # 생성된 설정 파일 목록 출력
    echo "생성된 설정 파일 목록:"
    for config_file in "sys_{syscalls[@]}"; do
        echo "$config_file"
    done
    ```
2.쉘 스크립트 실행.

    ./test.sh
3. sys_sysname파일 생성 완료
