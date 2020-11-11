# redux-thunk, multiparty, axios를 이용하여 원격 저장소(서버)에 파일 업로드

redux를 사용하고 비동기로 file upload하기 위해 [redux-thunk](https://github.com/reduxjs/redux-thunk)를 사용한다.

자세한 흐름은 아래와 같다.

- 클라이언트
    1. 버튼 클릭
    2. 파일 업로드 요청 action dispath
    3. axios post를 이용한 파일 업로드
- 서버
    1. multiparty를 이용해 클라이언트에서 받은 요청을 remote 저장소로 업로드

추가로, 요청할 때 progress도 요청한다.

- 클라이언트
1. 버튼 클릭 및 2. 파일 업로드 요청 action dispath

    Upload Component에 아래와 같은 handler를 추가해 준다.

    ```jsx
    class Upload extends React.Component {
      constructor(props) {
        super(props);
        this.onButtonClickHandler = this.onButtonClickHandler.bind(this);
      }
      onButtonClickHandler() {
        if (this.hasFile()) {
          this.props.reqUploadFiles(this.props.files);
        }
      }
      ...
      render() {
        return (
          ...
          <Button size="lg" onClick={this.onButtonClickHandler}>
            Upload
          </Button>
          ...
        );
      }
    }
    ```

    여기서 `this.props.reqUploadFiles`는 [redux connect](https://deminoth.github.io/redux/basics/UsageWithReact.html)로 연결해준 action이다. `this.props.files`는 input type="file"에서 넘겨 받은 file object 리스트가 담겨있다.
    redux에 관해 더 궁금한 사항이 있으면 [링크](https://deminoth.github.io/redux/)를 참고하면 된다. 예제도 많으니 참고

    reqUploadFiles의 코드는 아래와 같다.

    ```jsx
    const reqUploadFiles = files => async (dispatch) => {
      /* loading or progress를 출력하기 위함 */
      dispatch(uploadFilesPending(F.getTotalFileSize(files)));
      try {
        const res = await reqUploadFilesImpl(
          /* 전송할 파일 객체*/
          files,
          /* 파일 업로드의 progress를 state에 담기 위한 함수 */
          progressEvent => dispatch(uploadFilesPending(progressEvent.total, progressEvent.loaded)),
        );
        /* 파일 업로드가 성공했을 때, uploadFilesSuccess를 dispatch */
        return dispatch(uploadFilesSuccess({
          regiId: res.data.id,
          expireTime: res.data.expireTime,
        }));
      } catch (error) {
        /* 파일 업로드가 실패했을 때, uploadFilesFailure에 응답 코드를 담아  dispatch */
        return dispatch(uploadFilesFailure(error.response.status));
      }
    };
    ```

    여기서 `reqUploadFilesImpl` 함수는 실제 [axios](https://github.com/axios/axios)를 이용해서 서버와 통신하는 함수다.

2. axios post를 이용한 파일 업로드

    ```jsx
    const reqUploadFilesImpl = (files, onUploadProgress = F.emptyFunc) => {
      const url = C.API_URL.FILE;
      const formData = new FormData();
      files.forEach((file, i) => formData.append(['file', i].join(''), file));
      const config = {
        headers: {
          'content-type': 'multipart/form-data',
        },
        onUploadProgress,
      };
      return axios.post(url, formData, config);
    };
    ```

    formData에 file 객체를 담아 post 통신한다.

- 서버
    1. multiparty를 이용해 클라이언트에서 받은 요청을 remote 저장소로 업로드

        ```jsx
        import Multiparty from 'multiparty';
        import axios from 'axios';
        import FormData from 'form-data';

        import getConfig from '_modules/config';
        import Utils from '_modules/common/utils';

        const Config = getConfig();

        /* 클라이언트에서 POST 통신한 요청이 라우트에 의해 아래 함수로 온다*/
        const upload = (req, callback) => {
          /* Multiparty를 사용하여 업로드되는 파일을 받는다. 
          *  maxFilesSize로 최대 업로드 용량을 제한한다.
          */
          const form = new Multiparty.Form({ maxFilesSize: Config.tmpdir.file.maxSize });
          /* formData를 만들어 외부 저장소에 전송할 파일을 담는다 */
          let formData = new FormData();
          formData.maxDataSize = Infinity;
          let count = 0;

          form.on('part', (part) => {
            if (!part.filename) {
              part.resume();
            } else {
              /* 업로드한 파일을 하나씩 읽어 formData에 담는다*/
              formData.append(
                ['file', count].join(''),
                part,
                {
                  filename: part.filename,
                  contentType: part['content-type'],
                },
              );
              count += 1;
              part.resume();
            }
          });

          form.on('close', () => {
            const uploadConfig = Config.tmpdir.service.upload;
            const uploadUrl = Utils.getUrl(uploadConfig.hostname, uploadConfig.protocol, uploadConfig.port);
            const config = {
              headers: {
                accept: 'application/json',
                'Content-Type': `multipart/form-data; boundary=${formData.getBoundary()}`,
              },
              maxContentLength: Config.tmpdir.file.maxSize,
            };
            /* formData에 담은 파일을 외부 저장소에 POST로 전송 */
            axios.post(uploadUrl, formData, config)
            /* 외부 저장소에 업로드가 끝난 후 클라이언트로 다시 응답을 보낸다. */
              .then(res => callback(null, { code: res.status, data: res.data }))
              .catch(err => console.log('Axios post error: ', err));
          });
          form.on('error', (err) => {
            console.log('Multiparty form error', err);
            return callback(err);
          });
          form.parse(req);
        };
        ```
