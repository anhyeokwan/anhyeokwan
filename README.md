#  NSU ๊ณผํ APP ๐




- ### ํ๋ก์ ํธ ๊ฐ์



<img src="./READMEFILE/แแฉแแณแแฎแจ.png" width="70%" align-text:center >

<br>

  - โก [NSU ๋ฒ๊ฐ](https://www.nsu.ac.kr/)

  - `NSU ๊ณผํ APP`์ 2๋๋์์ ์ฝ๋ก๋ ์๊ตญ์์์ ์ธ๊ฐ๊ด๊ณ์ ๊ฒฐ์ฌ๋ฅผ ํํผํ๊ธฐ ์ํด NSU ํ์๋ค๋ง ์ฌ์ฉํ  ์ ์๋ WEB๊ธฐ๋ฐ์ APP์๋น์ค์ด๋ค.

<br>
<br>
<br>

- ### ์ฃผ์ ๊ธฐ๋ฅ 

  - **SSO ๋ก๊ทธ์ธ ์๋น์ค**

    > 1) ์ฐ๋ฆฌํ๊ต์ธ ๋จ์์ธ๋ํ๊ต **ํ์ํฌํ์ ๋ก๊ทธ์ธ์ด ๊ฐ๋ฅํ ์ฌ๋**๋ง ์ด์ฉํ  ์ ์๋๋ก ํ์ํฌํ์ ํ ํฐ์ ์ด์ฉํ์ฌ APP์ ๋ก๊ทธ์ธ ์๋น์ค ๊ตฌํ.
    >
    > 2)  ํ์๊ฐ์์ ๋ถํธํจ์ ๋๊ธฐ ์ํด, ํ์ํฌํ์์ ์ฌ์ฉํ๋ ํ๋ฒ๊ณผ ๋น๋ฒ์ ๊ทธ๋๋ก ์ฐ๋ฆฌ APP์์ ์ฌ์ฉ๊ฐ๋ฅ
    >
    > โ	- ์ฌ์ฉ์๋ ์ฐ๋ฆฌํ๊ต ํ์๋ค๋ก๋ง ์ด๋ค์ง ์ง๋จ์ด๊ธฐ๋๋ฌธ์, ์ฐ๋ฆฌ APP์ ๋ํ ์ ๋ขฐ๋๊ฐ ์ฆ๊ฐํจ.

<br>
<br>
<br>


   
* **[๊ธฐ์  ๊ตฌํ] :** ํ์ํฌํ๋ก๊ทธ์ธ์๋น์ค์ ๋ก๊ทธ์ธํ  ๋ ์์ด๋ ๋น๋ฒ ๊ฐ์ ์๋ ฅํ๋๋ฐ,
                  id, password param๊ฐ์ requestํด์ ๋์จ ์๋ต์ฝ๋ 10000์ ์ด์ฉํ์ฌ 
                  10000์ด ๋์ฌ ๊ฒฝ์ฐ ๋ค์ ํ์ด์ง๋ก ๋์ด๊ฐ๊ณ  ์๋ ๊ฒฝ์ฐ ๋ฆฌ๋ค์ด๋ ํธ ๋๋๋ก ๊ตฌํํ์๋ค.
                  





                  
```
@Controller
public class Certification {
   
   @Inject
   LoginService service;
   
   @Autowired
   private LoginService loginService;
   
   // ํ์๊ฐ์์ SSO ๋ก๊ทธ์ธ ๊ตฌํ
   @RequestMapping(value = "/NSUOK", method = { RequestMethod.GET, RequestMethod.POST })
    public String paramTest(
          HttpServletRequest request,
           @RequestParam("id") String id,
            @RequestParam("password") String password,
            Model model,
            RedirectAttributes rttr,
            MemberInfo member
            )
            throws Exception {
        // ํ์์์ ์ค์ ์ ์ํด HttpComponentsClientHttpRequestFactory ์ฌ์ฉ
        HttpComponentsClientHttpRequestFactory httpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        httpRequestFactory.setConnectTimeout(3000);
        httpRequestFactory.setReadTimeout(5000);
        
        
        // ์ธ์๊ฐ(id) ์ ์ง ํ ๋๊น. 
        HttpSession session = request.getSession();
        String name = id;
        session.setAttribute("id", name);
        ModelAndView mv = new ModelAndView();
        mv.setViewName(name);
        
        //์ธ์ฆํ์ธ
        
        String Class = request.getParameter("id");
        MemberInfo idCheck = loginService.idCheck(Class);
        
        int result1 = 0;
        
        if(idCheck != null) {
           result1 = 1;
        }

        HttpClient httpClient = HttpClientBuilder.create()
                .setMaxConnTotal(200)
                .setMaxConnPerRoute(20)
                .build();
        httpRequestFactory.setHttpClient(httpClient);
        // RestTemplate
        RestTemplate restTemplate = new RestTemplate(httpRequestFactory);
        // ํ๋ผ๋ฏธํฐ ์ค์ 
        MultiValueMap<String, String> parameters = new LinkedMultiValueMap<>();
        parameters.add("id", id);
        parameters.add("password", password);
        // POST ํธ์ถ
        String url = "https://sso.nsu.ac.kr/api/login";
        ResponseEntity<String> responseEntity = restTemplate.postForEntity(url, parameters, String.class);
        String body = responseEntity.getBody();
        ObjectMapper objectMapper = new ObjectMapper();
        Map result = objectMapper.readValue(body, Map.class);
        String resultCode = String.valueOf(result.get("code"));
        // ์๋ต ์ฝ๋ 10000 - ์ฑ๊ณต
        if (resultCode.equals("10000") && result1 == 0) {
            System.out.println("์ธ์ฆ ์ฑ๊ณต");
            service.potal(member);
            System.out.println(id);
            
            return "user/join";                       
        }if(resultCode.equals("10000") && result1 == 1) {
           System.out.println(result1);
           return "user/login";
           
        }else {
            System.out.println("์ธ์ฆ ์คํจ");
            rttr.addFlashAttribute("msg", false);
            return "redirect:/NSU";
        }

    }
   
   
   }

   ```
  
   

  - **๊ฒ์ํ ๋ฑ๋ก ๋ฐ ์ ์ฒญ** 

    > 1)  ์ฌ์ฉ์๋ค์ ์์ ์ ์์ํ ๊ฐ์ธ์ ๋ณด(๋์ด, MBTI, ์ฃผ๋)๋ฑ์ด ํฌํจ๋ ๊ฒ์๊ธ์ ์์ฑํ๋ค. 
    >
    > 2)  ๋ง์์ ๋๋ ๊ฒ์๊ธ์ ๋ค์ด๊ฐ์ ์น๊ตฌ์ ์ฒญ ๋ฒํผ์ ํด๋ฆญํ๋ค. 
    >
    
    
    
    

  - **์น๊ตฌ ์๋ฝ ๋ฐ ๋ฑ๋ก**

    > 1) ์น๊ตฌ์๋ฝ ์ ์ฒญ์ด ์ค๋ฉด, ์ฌ์ฉ์๋ ์๋ฝ์ด๋ ๊ฑฐ์ ๋ฒํผ์ ํ์ฉํ  ์ ์๋ค.
    >
    > 2) ์๋ฝ์ ๋๋ฌ ์๋ก ์น๊ตฌ์ํ๊ฐ ๋๋ฉด, ์น๊ตฌ ๋ชฉ๋ก์ ์ถ๊ฐ๋๋ค.
    >
    > 3) ์๋ก ์น๊ตฌ์ธ ์ํ์์๋ ์๋ก์ KAKAOTALK ID๋ฅผ ์๋์ผ๋ก ๊ตํํ  ์ ์๋ค.
    >

- ### ํฅํ ๊ณํ


  - **์ฌ์ฉ์ ํ๋กํ** : ํ์ฌ ์ฐ๋ฆฌ ํ๋ก์ ํธ์์ ๋ณด์ฌ์ง๊ณ  ์๋ ๋จ์ํ ํ๋กํ์ด ์๋ ์ฌ์ง์ด๋ ์ฌ๋ฌ ์์ค๋ค์ด ํฌํจ๋ ๋ค์ฑ๋ก์ด ํ๋กํ
  - **์ฑํ ๊ธฐ๋ฅ** : ์ฐ๋ฆฌ APP์์ KAKAOTALK ID๋ฅผ ๊ตํํด์ฃผ๋ ๋ฐฉ์์ด ์๋, APP์์ฒด์ ์ฑํ์๋น์ค๋ก ์ฌ์ฉ์๋ค๋ก ํ์ฌ๊ธ ํธ์๋ฅผ ์ ๊ณต
  - **๋ฐ์ดํฐ๋ฒ ์ด์ค ์ํธํ** : ์ฌ์ฉ์๋ค์ ์ ๋ณด๊ฐ ๋ฐ์ดํฐ๋ฒ ์ด์ค์ ๋ด๊ธธ ๋, BCyptPasswordEncoder์ ํ์ฉํ ์ํธํ ์์
  - **APP Design** : ์ฐ๋ฆฌ APP๋ง์ ์ ์ฒด์ฑ์ ์ง๋ ์ด์ฒด์  ๋์์ธ ๊ฐํธ 




## ๐ ๋ชฉ์ฐจ

[NSU ๊ณผํ APP ๐](#triangular_flag_on_post-run-with-me--%EF%B8%8F) 

* [์ฌ์ฉ๋ ๋๊ตฌ](#hammer_and_wrench-์ฌ์ฉ๋-๋๊ตฌ)

* [์๋น์ค ์๊ฐ](#-์๋น์ค-์๊ฐ)

* [์ ์](#-์ ์)

  


## :hammer_and_wrench: ์ฌ์ฉ๋ ๋๊ตฌ



![TechStack](https://user-images.githubusercontent.com/19357410/100544132-062d1380-3297-11eb-832e-9e1dd8f8da13.png)




**[ TEAM Cooperation ]**

- **GitHub** : GitHub์ ํ์ฉํ์ฌ ํ๋ก์ ํธ๋ฅผ ๊ด๋ฆฌ.
  - Git Flow ์ ๋ฐ๋ฅธ ๋ธ๋์น ์ ๋ต ์๋ฆฝ.
  
- **Slack** : ํ์์ ์ํ ๊ณต์ฉ ๋ฌธ์ ๋ฐ ์ฐ์ถ๋ฌผ๋ค์ ๊ณต์ ํ  ์ ์๋๋ก ํ์ฉ.
  - ๋์ ๋ฌธ์ ์์ฑ
  - ๋์ฉ๋ ํ์ผ ์ฒจ๋ถ

## ๐ ์๋น์ค ์๊ฐ

### 0. ํ์๊ฐ์ ํ๋ฉด

#### 0-1. ํ์๊ฐ์ ํ๋ฉด

<img src="./READMEFILE/ํ๊ต์ธ์ฆ ํ์๊ฐ์.jpg" width="30%"> <img src="./READMEFILE/ํ์๊ฐ์.jpg" width="30%"> 

- **[ํ์๊ฐ์ ํ๋ฉด] :** NSU ํ์ํฌํ์ ํ๋ฒ๊ณผ ๋น๋ฐ๋ฒํธ๋ฅผ ํ์ฉํ์ฌ ๋ณธ๊ต์์ธ์ง๋ฅผ ์ธ์ฆ ํ ์ธ์ฆ์ด ์๋ฃ๋ ํ์ฐ๋ค์๊ฒ๋ง ์ธ์ ์ฌํญ์ ๊ธฐ์ฌํ  ์ ์๋ ํ์ด์ง๋ก ๋๊ฒจ์ค๋ค.


### 1. ๋ก๊ทธ์ธ ํ๋ฉด

#### 1-1. ๋ก๊ทธ์ธ ํ๋ฉด

<img src="./READMEFILE/๋ก๊ทธ์ธํ์ด์ง.jpg" width="30%">



---

### 2. ๋ฉ์ธ ํ๋ฉด

#### 2-1. ๋ฉ์ธ ํ๋ฉด

<img src="./READMEFILE/๋ก๊ทธ์ธ ์  ํ.jpg" width="30%">  <img src="./READMEFILE/๋ก๊ทธ์ธ ํ ํ.jpg" width="30%">

- **[๋ฉ์ธ ํ๋ฉด] :** ๋ก๊ทธ์ธ ์ (๋ฉ์ธํ๋ฉด) / ๋ก๊ทธ์ธ ํ(๋ฉ์ธํ๋ฉด)

---

#### 2-2. ๋ฉ์ธ ํ๋ฉด์์ 1:1 ๋ฏธํ ํด๋ฆญ

<img src="./READMEFILE/๋ฑ๋ก๋ชฉ๋ก.jpg" width="30%"> <img src="./READMEFILE/๋ฑ๋กํ๊ธฐ.jpg" width="30%">  <img src="./READMEFILE/์์ธ๋ณด๊ธฐ.jpg" width="30%">

- **[๋ฉ์ธ ํ๋ฉด์์ 1:1 ๋ฏธํ ํด๋ฆญ์] :** ์ ์ ๋ค์ด ์์ฑํ ๊ฒ์๊ธ์ ๋ณด์ฌ์ค๋ค.
- **[๊ฒ์๊ธ์์ ๋ฑ๋กํ๊ธฐ ๋ฒํผ ํด๋ฆญ์] :** ์ ์ ์ ์์ํ ๊ฐ์ธ์ ๋ณด๋ค์ ์๋ ฅํ  ์ ์๋ ํผ์ ๋ณด์ฌ์ค๋ค.
- **[๊ฒ์๊ธ ํด๋ฆญ์] :** ๋ค๋ฅธ ์ ์ ๊ฐ ์๋ ฅํด๋์ ๊ฐ์ธ์ ๋ณด๋ค์ ํ์ธํ  ์ ์๋ ํ์ด์ง๊ฐ ๋ณด์ฌ์ง๋ค.
---

#### 2-3. ๋ง์์ ๋๋ ์ ์ ์๊ฒ ์น๊ตฌ์ ์ฒญ ๋ณด๋ด๊ธฐ

<img src="./READMEFILE/๋ฐ์์ ์ฒญ.jpg" width="30%"> <img src="./READMEFILE/์๋ฝ์๋ฃ.jpg" width="30%">

* **[๋ค๋ฅธ ์ ์ ์๊ฒ ์น๊ตฌ์ ์ฒญ์ ๋ฐ์ ๋] :** ๋ค๋ฅธ ์ ์ ๊ฐ ์น๊ตฌ ์ ์ฒญ๋ฒํผ์ ๋๋ฅด๋ฉด ์ ์ฒญ ํผ์ด ๋ ๋ผ์จ๋ค.
* **[๋ค๋ฅธ ์ ์ ์ ์น๊ตฌ์ ์ฒญ์ ์๋ฝํ  ๋] :** ์๋ฝํ ์น๊ตฌ์ ๋ชฉ๋ก์ ๋ณผ ์ ์๋ ํ์ด์ง๋ฅผ ๋ณผ ์ ์๋ค.

---

### 3. ํ๋กํ

#### 3-1. ํ๋กํ ์ค์ 

<img src="./READMEFILE/ํ2.jpg" width="30%">  <img src="./READMEFILE/์ฌ์ฉ์ํ๋กํ.jpg" width="30%">  <img src="./READMEFILE/ํ์ํํด.jpg" width="30%">

* **[ํ๋กํ ์ค์ ] :** ํ์๊ฐ์ ์ ๊ธฐ์ฌํ ์ธ์ ์ฌํญ ํ์ธ ๋ฐ ๋ณ๊ฒฝ.
* **[ํ์ ํํด] :** ํ์ฌ ๋น๋ฐ๋ฒํธ ์๋ ฅ ํ, ์๋ ฅํ ๋น๋ฐ๋ฒํธ๊ฐ ํ์ฌ์ ๋น๋ฐ๋ฒํธ์ ์ผ์นํ๋ฉด ํํด.


---

#### 3-2. ๊ด๋ฆฌ์ ๋ก๊ทธ์ธ ๋ฐ ํ์ ๋ชฉ๋ก

<img src="./READMEFILE/๊ด๋ฆฌ์๋ก๊ทธ์ธ.jpg" width="28.55%"> <img src="./READMEFILE/๊ด๋ฆฌ์๋ชฉ๋ก.jpg" width="30%"> <img src="./READMEFILE/ํ์๊ด๋ฆฌ์์ ์ญ์ .jpg" width="30%">


<img src="./READMEFILE/studentid.jpg" width="63%">

* **[๊ด๋ฆฌ์ ๋ชจ๋] :** Verify๊ฐ 9๋ก ์ง์ ๋ ๊ด๋ฆฌ์๋ชจ๋์์๋ ํ์๋ค์ ๊ด๋ฆฌ ์์  ์ญ์  ํ  ์ ์๋ค.

---


---


## ๐ค ์ ์

* ์ด๊ฑด - Lee Gun - LeeGun@naver.com - @[imdaeyong](https://github.com/imdaeyong) [Back/PL]
* ์ ๋ฏผ์ฐ - Jeon Min Woo - JeonMinWoo@gmail.com - @[kkmwkk](https://github.com/kkmwkk) [Back]
* ์ํ๊ด - An Hyeong Kwan - AnHyeongKwan@gmail.com - @[hyungtaik](https://github.com/hyungtaik) [Back]
* ์ ์ฌํ - Jeong Seung Pil - JeongSeungPil@gmail.com - @[LEESUNSOO](https://github.com/LEESUNSOO) [Front]



