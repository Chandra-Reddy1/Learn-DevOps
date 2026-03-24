# Shell Scripting Interview Questions & Answers (Top 20)

---

## 1. What is a shell script?

A shell script is a file containing a sequence of commands executed by a shell (bash/sh).

---

## 2. What is shebang?

First line of script:
#!/bin/bash  
It defines which interpreter to use.

---

## 3. How to run a shell script?

- chmod +x script.sh  
- ./script.sh  

OR  
- bash script.sh  

---

## 4. What are variables in shell?

Used to store values:
name="Chandra"  
echo $name  

---

## 5. What is difference between $ and ${}?

${var} is safer when concatenating variables:
echo ${name}_dev  

---

## 6. What is positional parameter?

Arguments passed to script:
$1, $2, $3  

---

## 7. What is $@ vs $*?

- $@ → treats arguments separately  
- $* → treats as single string  

---

## 8. What is exit status?

- 0 → success  
- non-zero → failure  

---

## 9. What is if condition?

```bash
if [ $a -eq 10 ]; then
  echo "Equal"
fi
