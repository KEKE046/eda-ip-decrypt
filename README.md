# eda-ip-decrypt
IEEE1735 decrypt script

```bash
decrypt() {
    key_name=$1
    file=$2
    out=$3

    temp=$(mktemp)

    key=$(awk "/$key_name/,/^$/" $file\
        | awk '/key_block/,/^$/' \
        | sed '1d' | base64 -d \
        | openssl rsautl -decrypt -inkey $key_name.pem \
        | od -t x1 -An | sed 's/\s//g')

    awk "/data_block/,/^$/" $file \
        | sed '1d' | base64 -d > $temp

    iv=$( dd if=$temp iflag=count_bytes count=16 \
        | od -t x1 -An | sed 's/\s//g')

    dd if=$temp iflag=skip_bytes skip=16 \
        | openssl aes-128-cbc -d -K $key -iv $iv > $out

    rm -rf $temp
}

```
