export from qiita

```bash
$ wget $(curl -s 'https://qiita.com/api/v2/users/ユーザID/items&page=1' | jq -r '[.[].url+".md"]|join(" ")')
```

* ref: https://qiita.com/Trouble_SUM/items/d14ec5abae867b9e5a6b

```bash
for i in ../../qiita-export/x/*.md; do filename=$(grep -m 1 title: ${i} | cut -f2 -d ' '); cp ${i} ${filename}.md ;done
```


slug

prename 's/[^a-zA-Z0-9.]+/-/g; s/-\././'  *

