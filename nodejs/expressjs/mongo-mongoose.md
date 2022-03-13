# MongDB

## 설치

> 개발환경은 WSL이다.

먼저 Microsoft 문서에 나와있는대로 WSL에 MongoDB 설치하기를 따라하자. [링크](https://docs.microsoft.com/ko-kr/windows/wsl/tutorials/wsl-database#install-mongodb)

다음 커맨드를 순서대로 입력하자. (버전은 추후 변경될 수 있으므로 링크에 나와있는대로 하도록)

```bash
# 1
cd ~

# 2
sudo apt update

# 3
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -

# 4
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list

# 5
sudo apt-get update

# 6
# mongodb-org 가 아닌 mongodb
sudo apt-get install -y mongodb
```

여기까지 했으면 MongoDB 설치는 끝났다.

이후의 과정은 MongoDB 실행하는 과정인데, Microsoft 문서에서 설명된 경로와 조금 다르니 다음을 참고하자. [링크](https://hell-of-company-builder.tistory.com/35)

간략히 말하자면 `data/db`의 경로가 다른데, 그 이유는 mongoose를 사용하기 위함이라고 생각하면 된다.

```bash
# 7
# ~/data/db 가 아닌 /data/db
mkdir -p /data/db

# 8
sudo chown -R `id -un` data/db

# 9
sudo service mongodb start

# 10
# mongo 입력 후 mongo shell 이 실행되면 완료!
mongo
```

## 실행

mongodb를 사용하려면 다음의 커맨드를 이용하여 인스턴스를 실행해줘야 한다.

```bash
sudo service mongodb start # 시작
sudo service mongodb stop # 종료
sudo service mongodb status # 상태 확인
```

커맨드가 살짝 기니까 alias를 이용하자. (`~./.zshrc` 또는 자신의 터미널 프로필)

```.zshrc
# alias
alias mongo-start="sudo service mongodb start"
alias mongo-stop="sudo service mongodb stop"
alias mongo-status="sudo service mongodb status"
```

# Mongoose

## 설치

프로젝트에 npm으로 mongoose 설치

```bash
npm i mongoose
```

## 설정

```js
import mongoose from 'mongoose';

mongoose.connect('mongodb://127.0.0.1:27017/wetube');

const db = mongoose.connection;

db.on('error', (error) => console.log('DB Error', error));
db.once('open', () => console.log('✅ Connect to DB 🎉'));
```

### connect uri

connect uri는 mongodb 인스턴스를 실행한 후 터미널에 `mongo`를 입력하면 나오는 uri를 이용하면 된다.

uri 뒤에 있는 `wetube`는 db명이다. 명시적으로 입력을 해 놓으면 생성을 따로 하지 않아도 된다.

## Schema

Schema는 db에 저장될 오브젝트의 형식을 지정하는 것이다.

필드, 타입, 어트리뷰트등을 통해 세부사항을 정할 수 있다.

해당 schema로 모델을 만들어 CRUD를 실행할 수 있다.

```js
import mongoose from 'mongoose';

const videoSchema = new mongoose.Schema({
  title: { type: String, required: true, trim: true, maxlength: 80 },
  description: { type: String, required: true, trim: true, minlength: 20 },
  createdAt: { type: Date, required: true, default: Date.now },
  hashtags: [{ type: String, trim: true }],
  meta: {
    views: { type: Number, default: 0 },
    rating: { type: Number, default: 0 },
  },
});

const Video = mongoose.model('Video', videoSchema);

export default Video;
```

## middleware

middleware 함수는 특정한 기능을 실행할 때 추가적인 처리를 할 수 있도록 도와주는 역할을 한다.

`this`를 통해 오브젝트에 접근하는데, 이때 콜백함수는 화살표함수로 하면 안된다. this가 달라지기 때문.

```js
videoSchema.pre('save', async function () {
  this.hashtags = this.hashtags[0].split(',').map((hashtag) => (hashtag.startsWith('#') ? hashtag : `#${hashtag}`));
});
```

## static

schema model에 추가적인 함수를 장착해주는 기능.

아래의 static 함수는 `Video.formatHashtags(hashtags)`로 호출할 수 있다.

```js
videoSchema.static('formatHashtags', (hashtags) => hashtags.split(',').map((hashtag) => (hashtag.startsWith('#') ? hashtag : `#${hashtag}`)));
```

## find filtering

find를 할 때 단순히 필드에 값을 할당할 수 있지만, 그렇게 되면 정확히 일치하는 결과만 가져오게 된다.

다양한 조건들을 이용하고 싶다면 오브젝트형태로 filter할 수 있는 기능이 제공된다.

```js
const videos = await Video.find({
  title: {
    $regex: new RegExp(keyword, 'i'),
  },
});
```
