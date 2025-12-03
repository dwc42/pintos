clone to /pintos
make sure the repo is not /pintos/pintos

**Design Document** 
```bash
pintos/design_document.md
```



## ✅ Steps To clone the repo without crashing the os. (From chat so might not work).

### 0. Open a terminal
open a terminal in the pintos/ directory

### 1. Check current remotes

```bash
git remote -v
```

You will probably see:

```
origin  https://github.com/Stanford.../pintos.git
or
origin  https://github.com/fghanei/pintos.git
```

### 2. Change `origin` to point to **your fork**

```bash
git remote set-url origin https://github.com/dwc42/pintos.git
```

### 3. Verify it worked

```bash
git remote -v
```

Should now show:

```
origin  https://github.com/dwc42/pintos.git
```

### 4. Pull your latest changes

```bash
git pull origin master
```

### 5. Open vscode

```bash
code .
```


