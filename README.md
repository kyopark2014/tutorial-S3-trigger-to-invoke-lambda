# tutorial-S3-trigger-to-invoke-lambda

이 문서에서는 S3에 파일 전송시 Lambda가 실행하여 파일정보를 전달하는 동작을 설명하겠습니다. 

## Lambda 

1. Lambda console로 접속하여 [Functions] - [Create functions] 선택합니다.

<img width="1360" alt="image" src="https://user-images.githubusercontent.com/52392004/154180390-7a43fb3f-22c0-4533-b17f-2c4c4f385ed8.png">

2. [Function name]으로 “S3-event-trigger”라고 입력하고, 나머지는 기본 설정으로 [Create function]을 선택합니다. 

![image](https://user-images.githubusercontent.com/52392004/154180792-ee982074-1fd3-43be-894f-0e13e855f216.png)

3. Lambda가 S3에 접속할 수 있도록 퍼미션을 설정하기 위하여 아래와 같이 [Configuration] - [Permissions]에서 Role name인 “S3-event-trigger-role-wwjkfubg”을 선택합니다.

<img width="861" alt="image" src="https://user-images.githubusercontent.com/52392004/154182641-7a738c8c-4da2-44e6-af5f-d125ab21d9f9.png">

4. [IAM] - [Roles] - [S3-event-trigger-role-wwjkfubg]로 이동되면, [Permissions]의 [Policy name]에서 “AWSLambdaBasicExecutionRole-057780e6-ca12-4fb7-a7c0-dd604c9bd4e7”을 선택합니다.

<img width="860" alt="image" src="https://user-images.githubusercontent.com/52392004/154183037-f2feb24f-825c-4af8-b3d0-0b01886040cf.png">

5. 여기서 S3에 대한 퍼미션을 추가하기 위해 Edit policy를 선택합니다.


<img width="855" alt="image" src="https://user-images.githubusercontent.com/52392004/154183207-0a7636cd-3f82-4bf0-ad75-ff7d1d310bbd.png">

6. S3 퍼미션을 추가 합니다.

추가하여야 할 퍼미션은 아래와 같습니다. 여기서, “simple-file-store”은 S3에 생성할 Bucket 이름입니다. 

```c
{
          "Effect": "Allow",
          {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:GetBucketLocation",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::simple-file-store/*"
            ]
        }
}
```

아래와 같이 추가후에 [Review policy]를 선택합니다.

<img width="869" alt="image" src="https://user-images.githubusercontent.com/52392004/154210321-900fce67-5270-43f0-b3e6-a813f0246274.png">





아래와 같이 기존의 CloudWatch Logs이외에 S3 퍼미션이 잘 입력되었는지 확인후 [Save changes]를 선택합니다.



![image](https://user-images.githubusercontent.com/52392004/154210514-b453fd25-bd51-4fcc-8526-e25d614ab9c3.png)


7. [S3-event-trigger] - [Code]로 이동해 아래 코드를 붙여넣기를 하고, [Deploy]선택하여 Lambda에 코드를 반영합니다. 


<img width="1060" alt="image" src="https://user-images.githubusercontent.com/52392004/154214798-6d1740f3-fd4e-4ddb-aaff-d926486d2e7a.png">

```c
const aws = require('aws-sdk');

const s3 = new aws.S3({ apiVersion: '2006-03-01' });

exports.handler = async (event, context) => {
    console.log('## ENVIRONMENT VARIABLES: ' + JSON.stringify(process.env))
    console.log('## EVENT: ' + JSON.stringify(event))

    // Get the object from the event and show its content type
    const bucket = event.Records[0].s3.bucket.name;
    const key = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, ' '));
    const params = {
        Bucket: bucket,
        Key: key,
    }; 

    try {
        const { ContentType } = await s3.getObject(params).promise();
        console.log('CONTENT TYPE:', ContentType);
    } catch (err) {
        console.log(err);
    }

    const fileInfo = {
        Bucket: bucket,
        Name: key,
        Size: event.Records[0].s3.object.size
    }; 
    console.log('file info: ' + JSON.stringify(fileInfo))

    const response = {
        statusCode: 200,
        body: fileInfo,
    };
    return response;
};

```

## S3

1. S3 console의 [Buckets]에서 [Create bucket]을 선택합니다.

https://s3.console.aws.amazon.com/s3/home?region=us-west-2



![image](https://user-images.githubusercontent.com/52392004/154184198-6623afee-85fb-4f7d-ae8c-7c98bd567f28.png)

2. [Create bucket]에서 [Bucket name]은 “simple-object-store”로 입력하고 [AWS Region]을 “Asia Pacific (Seoul) ap-northeast-2”을 선택합니다. 나머지는 모두 기본값으로 유지하고, 아래로 스크롤하여 [Create bucket]을 선택합니다.




![image](https://user-images.githubusercontent.com/52392004/154200949-817a906c-f2a9-46bb-9d0a-d0d16affe5cb.png)


3. 다시 [Amazon S3] - [simple-file-store] - [Properties]로 이동한다.  


![image](https://user-images.githubusercontent.com/52392004/154201606-9a97a8d8-fca8-4dc9-913e-390d07f52b40.png)

이때 스크롤하여 [Event notifications]에서, [Create event notification]을 선택한다.

![image](https://user-images.githubusercontent.com/52392004/154201564-d81c8d85-8ed9-4ea5-b9d9-315c8dfd3483.png)


4. [Create event notification]에서 [Event name]은 “new-arrival-event”로 입력 합니다.


![image](https://user-images.githubusercontent.com/52392004/154202236-e50c054e-87c7-4355-bccd-35429df8cb59.png)

5. 하단으로 스크롤하여 [Event types]에서 [Object cretion]에서 [Put]을 아래와 같이 선택합니다. 이렇게 되면 새로운 파일을 S3로 업로드시에 Event가 발생합니다.


![image](https://user-images.githubusercontent.com/52392004/154202341-93520bce-9b5f-4883-becb-10e37f362bde.png)

6. 다시 아래로 스크롤하여 [Destination]에서 [Lambda function]을 선택하고, function으로 이미 만들어놓은 "S3-event-trigger]를 선택한다음에 [Save changes]를 통해 event를 등록합니다. 
![image](https://user-images.githubusercontent.com/52392004/154477821-7b01c66a-7d8c-4f36-94bb-5a437ceb2fe8.png)

7. [Amazon S3] - [simple-file-store] - [Objects]의 [Upload]를 선택하여 파일을 업로드 합니다.

![image](https://user-images.githubusercontent.com/52392004/154204326-91a27c4e-c7df-4099-a72c-31647ca336d2.png)

8. 여기에서는 아래와 같이 “sample2.jpeg”을 업로드 하였습니다 .


![image](https://user-images.githubusercontent.com/52392004/154204518-e0120bdc-1a61-4d23-831a-3dc9536909b0.png)

9. 아래와 같이 파일이 올라간것을 확인합니다.


![image](https://user-images.githubusercontent.com/52392004/154204667-407258e9-b87d-4aeb-b95a-60efdf497695.png)

10. CloudWatch로 이동하여 [Logs] - [Log groups]에서 아래와 같이 “/aws/lambda/S3-event-trigger”가 생성된것을 확인 할 수 있습니다. 해당 log를 선택하여 들어갑니다.

https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups



![image](https://user-images.githubusercontent.com/52392004/154212646-c0435224-00e9-43e6-8d1e-a6a6d72581aa.png)


11. 아래와 같이 해당 로그로 들어가 실행결과를 확인 할 수 있습니다.



<img width="1159" alt="image" src="https://user-images.githubusercontent.com/52392004/154212888-8e2ef819-687a-4902-8da5-fff44b2dc69a.png">


Event 로그를 보면 아래와 같은 정보들을 알 수 있습니다.

```c
INFO	## EVENT: {
    "Records": [
        {
            "eventVersion": "2.1",
            "eventSource": "aws:s3",
            "awsRegion": "ap-northeast-2",
            "eventTime": "2022-02-16T06:55:37.368Z",
            "eventName": "ObjectCreated:Put",
            "userIdentity": {
                "principalId": "AWS:AIDAZ3KIXN5TCGEFNC73W"
            },
            "requestParameters": {
                "sourceIPAddress": "54.239.119.1"
            },
            "responseElements": {
                "x-amz-request-id": "FX62B5YKY9TEA9AM",
                "x-amz-id-2": "pL9tG+q/lku/fbyhmXHpwVE4SjZh/3VRE7bUVvj2tPnofNZ+4dnvKqLU/NFUTq+N6fSfNtTuuTp6h0Gn2m+ITI7x90jxaCWGeiSM7jzvxds="
            },
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "new-arrival-event",
                "bucket": {
                    "name": "simple-file-store",
                    "ownerIdentity": {
                        "principalId": "AIBJOHCGWLN"
                    },
                    "arn": "arn:aws:s3:::simple-file-store"
                },
                "object": {
                    "key": "sample2.jpeg",
                    "size": 2073513,
                    "eTag": "a4627a0408ec8378aa77065a44650843",
                    "sequencer": "00620C9FE940BE6B81"
                }
            }
        }
    ]
}

```

Bucket 및 Contents의 이름과 크기는 아래와 같이 로그로 확인 할 수 있습니다. 

```c
INFO	file info: {
    "Bucket": "simple-file-store",
    "Name": "sample2.jpeg",
    "Size": 2073513
}
```




Reference 

https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html

https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html
