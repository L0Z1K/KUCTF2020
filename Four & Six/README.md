# Four & Six

주어지는 파일 : `problem.zip` 

```
password is "open".
```



**Flag** : `KOREA{c237d0ee1d374bc6c60c2da4269533aa9c13fc00595753618cd93a2ad0f41f87}`



압축파일을 그냥 풀게 되면, `flag.docx` 라는 파일이 나온다. `flag.docx` 에는 플래그가 없으므로 넘어간다.

**떡밥 1.** zip file size랑 압축해제된 docx 파일 size 차이가 크다.

**떡밥 2.** `$ strings problem.zip` 시 `image.png` 라는 스트링이 보인다.



### Intended solution

```
1. Recover local file header
2. binwalk
```

압축 파일을 분석해보면,

**End of Central Directory**

<img width="429" alt="image" src="https://user-images.githubusercontent.com/64528476/95415624-b9e6e680-096b-11eb-9ace-11b6fe6c314a.png">

- end of central dir signature : `504B0506`
- total entry : 2
- size of the central directory : 0xB6

End of Central Directory 헤더 부분을 보면, Total entry가 2다. 즉, 어떤 파일이 제대로 압축해제가 안된 것이다.

**Central Directory**

<img width="640" alt="image" src="https://user-images.githubusercontent.com/64528476/95415711-ff0b1880-096b-11eb-80a5-fc5cc860908e.png">

- signature : `504B0102`
- version made by : 0x033F
- version needed to extract : 0x0314
- general purpose bit flag : 0x0001
- compression method : 0x0008
- last mod file time : 0x6a9c
- last mod file date : 0x5148
- crc-32 : 0x3A1CE9BE
- compressed size : 0x3348C1
- uncompressed size : 0x334E17
- file name length : 0x09
- file name : `image.png`

실제로 Central Directory는 존재하는 데 `image.png` 에 대한 Local File Header가 조회가 되지 않는다. Local File header를 다시 복구해주어야 한다.

local file header offset이 0x2dcd = 11725임을 알 수 있고, 따라서 offset 11725 부분에 다음과 같은 헤더를 insert 해준다.

**Local file header**

- signature : `504B0304`
- version needed to extract : 0x0314
- general purpose bit flag : 0x0001
- compression method : 0x0008
- last mod file time : 0x6a9c
- last mod file date : 0x5148
- crc-32 : 0x3A1CE9BE
- compressed size : 0x003348C1
- uncompressed size : 0x00334E17
- file name length : 0x0009
- extra field length : 0x0000
- file name : `image.png`

<img width="630" alt="image" src="https://user-images.githubusercontent.com/64528476/95430568-e5c39580-0986-11eb-8ab6-f4d3a57439db.png">



다시 압축 해제를 하면, `image.png` 가 나온다. `binwalk` 하면 파일 중간에 PNG 헤더를 발견할 수 있다. 푸터도 찾아 오프셋 2363296-2366107 데이터만 추출하면 QR code 이미지가 나오고 플래그를 발견할 수 있다.

---