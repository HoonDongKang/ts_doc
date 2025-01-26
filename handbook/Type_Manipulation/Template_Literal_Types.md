템플릿 리터럴 타입(`Template literal types`)는 문자열 리터럴 타입을 기반으로 하며 유니온을 통해 여러 문자열로 확장될 수 있는 기능을 제공합니다.

이들은 자바스크립트의 템플릿 리터럿 문자열과 동일한 구문을 가지지만 타입 위치에서 사용됩니다. 구체적인 리터럴 타입과 함께 사용될 때, 템플릿 리터럴은 내용을 연결하여 새로운 문자열 리터럴 타입을 생성합니다.

```ts
type World = "world";
 
type Greeting = `hello ${World}`;
        //type Greeting = "hello world"
```

유니온이 삽인되는 위치에 사용될 경우, 해당 타입은 유니온의 각 멤버로 표현이 가능한 모든 문자열 리터럴이 됩니다.

```ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
 
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
          //type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

템플릿 리터럴의 각 삽입 위치에 유니온들은 교차 곱셈(`cross multiplied`)됩니다.

```ts
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";
 
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
            //type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"
```

이 기능은 작은 문자열의 유니온들을 다룰 때 더 유용합니다.

### String Unions in Types