# AWS_S3_Multer_Method

```javascript
AWS.config.update({
    accessKeyId: process.env.S3_ACCESS_KEY_ID,
    secretAccessKey: process.env.S3_SECRET_ACCESS_KEY,
    region: 'ap-northeast-2',
});

const AWSUpload = multer({
    storage: multerS3({
        // 저장 위치
        s3: new AWS.S3(),
        bucket: 'chacarda-video',
        acl: "public-read",
        contentType: multerS3.AUTO_CONTENT_TYPE,
        key(req, file, cb) {
            const ext = path.extname(file.originalname);
            const idx = req.body.idx;
            cb(null, dayjs(Date.now()).format("YYYYMMDD") + '/' + idx + '/' + path.basename(file.originalname , ext) + ext);
        }
    }),
    fileFilter: async function (req, file, cb) {
        const param = req.body;
        if(param != null) {
            if (param.idx == null){
                req.paramiterError = "입력되지 않은 파일 정보입니다. 요청할 수 없습니다.";
                return cb(null, false , new Error("입력되지 않은 파일 정보입니다. 요청할 수 없습니다."))
            }
            const result = await videoFileService.videoFileExistParameter(param.idx);
            if (!result){
                req.paramiterError = "파일 파라미터 정보가 존재하지 않습니다. 파일 업로드를 진행할 수 없습니다.";
                return cb(null, false , new Error("파일 파라미터 정보가 존재하지 않습니다. 파일 업로드를 진행할 수 없습니다."))
            }

            const resultCount = await videoFileService.videoFileCount(param.idx);
            if (resultCount){
                req.paramiterError = "이미 같은 파라미터 정보를 가진 파일이 업로드 되어 있습니다.";
                return cb(null, false , new Error("이미 같은 파라미터 정보를 가진 파일이 업로드 되어 있습니다."))
            }

            cb(null, true)
        } else {
            return cb(null , false)
        }
    }
}).single("file");
```
