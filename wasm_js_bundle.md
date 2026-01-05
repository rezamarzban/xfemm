```
wget [https://github.com/rezamarzban/xfemm/raw/refs/heads/master/femm_arm64_bin.tar.gz](https://github.com/rezamarzban/xfemm/raw/refs/heads/master/wasm_js_bundle.tar.gz)
tar -xzvf femm_arm64_bin.tar.gz -C .

node fmesher/fmesher.js problem.fem
node fsolver/fsolver.js problem
```
