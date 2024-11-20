## 배포 다이어그램

![배포다이어그램](https://github.com/user-attachments/assets/c04b1518-b125-41bb-8e24-0de6eadbaa93)

## 주요 링크

- S3 버킷 웹사이트 엔드포인트:[http://hanghaebucket.s3-website-us-east-1.amazonaws.com](http://hanghaebucket.s3-website-us-east-1.amazonaws.com/)
    <details>
        <summary>페이지 미리보기</summary>
        
    ![image](https://github.com/user-attachments/assets/1526bedb-6493-47ec-aa52-010504a393c8)
    </details>
- CloudFrount 배포 도메인 이름: [https://dlals0b0r7y3w.cloudfront.net](https://dlals0b0r7y3w.cloudfront.net/)
    <details>
        <summary>페이지 미리보기</summary>

    ![image](https://github.com/user-attachments/assets/32e3370f-63a3-4275-8150-3a8093c28424)
    </details>

<cloud front                                               / s3>

![image](https://github.com/user-attachments/assets/6cec4e5e-0e3a-4af6-8ce9-e8f9b1b8b3b4)

## 주요 개념

- <details>
  <summary>GitHub Actions과 CI/CD 도구:</summary>
  GitHub Actions: GitHub에서 제공하는 자동화 도구로 리포지토리에서 발생하는 push,pull,pr 등의 엑션에 따라 워크플로우(yml파일)을 자동으로 실행할 수 있습니다. </br>
  CI/CD(Continuous Integration/Continuous Deployment) 도구: 코드를 자동으로 빌드, 테스트, 배포할 수 있게 해주어 소프트웨어 개발 주기가 효율적으로 관리될 수 있습니다. </br>
    </br>
  결론적으로 GitHub Actions이 이러한 CI/CD 프로세스를 자동화 및 관리의 역할을 해줍니다. (깃헙 짱)
     </br>
   <ins>해당 repo의 상태를 예로 들어, 새로운 코드가 푸시되면 GitHub Actions가 자동으로 빌드를 실행하고 S3 및 CloudFront에 배포.  </br></ins>
   * 공유된 yml파일 내 node version을 명시하지 않고 있어 해당 부분 추가하였습니다.
     </br>
</details>

- <details>
  <summary>S3와 스토리지:</summary>
    객체 스토리지 서비스 중 aws에서 제공하는 제품명이 Amazon S3(Simple Storage Service) </br>
    (azure에서는 azure storage 서비스가 있음.) </br>
  -> 객체 스토리지는 데이터를 안전하게 저장하고 어디에서나 액세스할 수 있게 해줍니다. 주로 정적 웹 사이트 호스팅, 백업, 데이터 아카이빙 등에 사용됩니다. </br>
    •	무제한 스토리지 용량 </br>
    •	고가용성 및 내구성 </br>
    •	정적 콘텐츠(HTML, CSS, 이미지 등) 호스팅 가능 </br>
  <ins>해당 repo에서는 정적 파일과 리소스를 s3에 저장하고 CloudFront(*aws에서 제공하는 CDN서비스)와 연계하여 최적화하여 배포.</ins>
    </br>
</details>

- <details>
  <summary>CloudFront와 CDN:</summary>
  CloudFront: AWS에서 제공하는 콘텐츠 전송 네트워크(CDN) 서비스입니다. </br>
  CDN(Content Delivery Network): 사용자의 물리적 위치에 따라 가장 가까운 서버에서 콘텐츠를 제공하여 지연 시간을 줄이고 성능을 최적화하는 시스템입니다. (피자 가맹점과 배달부)</br>
  전 세계 엣지 로케이션을 통해 웹사이트, 동영상, API 등의 콘텐츠를 빠르고 안전하게 전송합니다.</br>
    •	캐싱: 자주 요청되는 콘텐츠를 캐싱하여 전송 속도 개선</br>
    •	HTTPS 지원: 안전한 데이터 전송 보장</br>
    •	S3와 연계: S3에서 호스팅되는 콘텐츠를 효율적으로 제공</br>
  Q: s3(스토리지)가 있어야 cdn을 사용할 수 있나?</br>
  A: cdn은 기본적으로 원본 파일을 저장할 스토리지가 있어야하겠지만, </br>
     1. 서버리스 cdn: cloudflare Worker,LambdaEdge같은 서비스 사용시 스토리지 없이 엣지에서 콘텐츠를 동적으로 생성/변경 가능 </br>
     2. 프록시 CDN 설정: 스토리지를 기존 서버에서 관리(오리진)하고 , cdn을 캐싱 레이어로만 활용가능. </br>
     3. in-memory 캐싱: cdn 엣지 로케이션에서 메모리기반 캐싱 사용. but,영구 스토리지가 없어 무효화 시 원본에서 다시 가져와야함. </br>
    </br>
</details>

- <details>
  <summary>캐시 무효화(Cache Invalidation): </summary>
     CDN 서비스는 콘텐츠를 엣지 서버에 캐싱하여 빠르게 제공하지만, 업데이트된 파일은 즉시 반영되지 않을 수 있습니다. 그래서 캐시를 무효화화여 이전 버전의 캐싱된 파일을 제거하고 최신 파일을 제공하도록 CDN을 업데이트하는 과정입니다.</br>
    <ins>해당 repo의 github 엑션을 돌리고나니 자동으로 무효화가 진행되었는데, 그 이유가 제공된 yml 파일의 아래 코드 때문이였다.</ins></br>

       run: |
            aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"

    aws에서도 무효화 설정을 할 수 있는데 , </br>
    
    1.	**AWS Management Console에 로그인**
    •	[AWS Management Console](https://aws.amazon.com/console/)로 이동하여 로그인합니다.
    
    2.	**CloudFront 서비스로 이동**
    •	상단 검색창에 CloudFront를 입력하거나 **Services** 메뉴에서 **CloudFront**를 선택합니다.
  
    3.	**배포 선택**
    •	**Distributions** 탭에서 캐시 무효화를 수행할 CloudFront 배포(Distribution)를 클릭합니다.
    
    4.	**Invalidations 생성**
    •	배포 상세 페이지에서 **Invalidations** 탭으로 이동합니다.
    •	**Create Invalidation** 버튼을 클릭합니다.

    6.	**파일 경로 입력**
    •	무효화하려는 파일 경로나 패턴을 입력합니다.</br>
    •	특정 파일: /path/to/your/file.html</br>
    •	전체 경로(모든 파일): /*</br>
    •	특정 폴더: /static/*</br>
    
    경로는 **CloudFront에서 접근 가능한 URL 경로**를 기준으로 작성해야 합니다. (예: S3의 파일 경로와 일치)
    
    6.	**요청 제출**
    •	입력을 완료한 후 **Invalidate** 버튼을 클릭합니다.
    
    7.	**무효화 상태 확인**
    •	Invalidation 요청이 생성되고 **In Progress** 상태로 표시됩니다.
    •	몇 분 정도 걸린 후 **Completed** 상태가 되면 무효화가 완료됩니다.
  </br>

  "사실, 최대한 캐시 무효화를 최소화하는 것이 좋다. (매달 1,000개 요청까지는 무료입니다. 그 이후에는 추가 요금)</br>
    •	캐시 무효화는 비용이 발생하므로 파일 이름에 버전 정보를 추가(예: app-v2.css)하여 캐시를 제어하는 방식을 사용하는 것이 권장"

</details>

- <details>
  <summary>Repository secret과 환경변수: </summary>
   Repository secret: GitHub에서 민감한 정보를 안전하게 저장하여, 워크플로우에서 사용할 수 있도록 제공하는 기능 (github의 env 파일?ㅎㅎ) </br>
   환경변수: 배포 및 실행 환경에 따라 동적으로 값을 설정하는 변수</br>
    <해당repo에서 아래와 같이 설정 > 
        
    ![해당repo](https://github.com/user-attachments/assets/5a6841d2-3c2e-42f7-9321-9996d69cd4b4)

</details>


##

<details>
  <summary>next.js default README</summary>
  
## Getting Started

This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.

</details>

