```
wget https://github.com/rezamarzban/xfemm/raw/refs/heads/master/wasm_js_bundle.tar.gz
tar -xzvf wasm_js_bundle.tar.gz -C .

node fmesher/fmesher.js problem.fem
node fsolver/fsolver.js problem
```
