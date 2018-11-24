# User HTTPS

## When starting from the beginning

```shell
openssl genrsa -out ./keys/key.pem 4092
openssl rsa -in ./keys/key.pem -outform PEM -pubout -out ./keys/public.pem

npm install -g pem-jwk

cat ./keys/public.pem | pem-jwk > ./keys/public.jwk.json
```

## When adding a new rule:

```shell
./make-trivial-rule {domain.com}
```

Then

```shell
python3 merge-rulesets.py

git add ./rules/*
git commit -m "Add new rule"

rm -f ./rulesets/v1/*.*
rm -f ./rulesets/v1/latest-rulesets-timestamp

./standalone.sh ./keys/key.pem ./rulesets/v1/

git add ./rulesets/v1/*
git commit -m "Update database ruleset"

git push -u origin master
echo Done
```
