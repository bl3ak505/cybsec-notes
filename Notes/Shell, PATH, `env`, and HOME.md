# Shell, PATH, `env`, and HOME — Clean Notes

---

## 1. How commands run

When you type:

```bash
wireshark
```

Flow:

```
Shell → checks PATH → finds /usr/bin/wireshark → executes it
```

Actual execution:

```
execve("/usr/bin/wireshark", args, env)
```

---

## 2. PATH (Command Lookup System)

```bash
echo $PATH
```

Example:

```
/usr/local/bin:/usr/bin:/bin
```

- Ordered list of directories
    
- Shell searches **left → right**
    
- First match is executed
    

---

### Types of execution

|Command|Behavior|
|---|---|
|`wireshark`|Found via PATH|
|`/usr/bin/wireshark`|Direct execution (no PATH)|
|`./script.sh`|Run from current directory|

---

### Key rule

```
Current directory (.) is NOT in PATH
```

---

## 3. Environment Variables (`env`)

View all:

```bash
env
```

They are:

```
Key = Value pairs passed to programs
```

Important ones:

```
HOME → your home directory
PATH → command search paths
USER → username
```

---

### `env` usage

Run with custom variable:

```bash
env MYVAR=test bash -c 'echo $MYVAR'
```

Run with clean environment:

```bash
env -i bash
```

---

## 4. `export` vs normal variable

```bash
MYVAR=hello
```

- Local to shell
    

```bash
export MYVAR=hello
```

- Passed to programs (environment variable)
    

---

## 5. HOME variable

```bash
echo $HOME
```

Example:

```
/home/bleak
```

---

### `cd` behavior

```bash
cd
```

is equivalent to:

```bash
cd $HOME
```

---

## 6. Changing HOME

Temporary:

```bash
export HOME=/tmp/myhome
```

Now:

```bash
cd
```

→ goes to `/tmp/myhome`

---

### Important

Directory must exist:

```bash
mkdir -p /tmp/myhome
```

---

### Effects of changing HOME

- Config files move (`~/.config`)
    
- History changes
    
- Tools (like zoxide) may break
    
- Programs trust HOME blindly
    

---

## 7. PATH manipulation

Add directory:

```bash
export PATH=/tmp:$PATH
```

Replace completely:

```bash
export PATH=/tmp
```

---

### Order matters

```
Front → high priority
End   → low priority
```

---

## 8. Creating custom commands

```bash
mkdir ~/bin
echo 'echo hello' > ~/bin/testcmd
chmod +x ~/bin/testcmd

export PATH=$HOME/bin:$PATH
testcmd
```

---

## 9. Shell built-ins

Check:

```bash
type cd
```

Example:

```
cd is a shell builtin
```

Why:

- Needs to modify shell state directly
    
- External program cannot change shell directory
    

---

## 10. Common mistakes

- Forgetting `./` for local scripts
    
- Breaking PATH → commands stop working
    
- Setting invalid HOME → system issues
    
- Confusing command vs argument:
    
    ```
    wireshark file → file is argument, not executable
    ```
    

---

## 11. Mental Models

### Command execution

```
command → PATH lookup → full path → kernel executes
```

### Environment

```
shell → passes env → program runs using it
```

### HOME

```
cd (no args) → goes to $HOME
```

---

## 12. Security Insights

- PATH hijacking → fake binaries executed first
    
- HOME manipulation → redirect configs/data
    
- Env abuse → influence program behavior
    

---

## One-line Summary

Shell finds commands using PATH, runs them via the kernel, and everything depends on environment variables like HOME and PATH.