# Spring Boot 파일 업로드 Unit Test

### 실 서비스

- 흐름

업로드 서비스는 현재 서비스가 위치한 서버와 다른 장비의 Storage와 Database에 파일, 파일정보를 저장하고 있다.

![./file-upload-service-flow.png](./file-upload-service-flow.png)

- File Upload Service

```java
@PostMapping("/")
/* MultipartFile로 request 받음 */
public ResponseEntity<FileInfo> uploadFile(@RequestParam("file") MultipartFile request) {
  FileInfo fileInfo = new FileInfo();
  fileInfo.setId(fileUploadService.createId());
  fileInfo.setName(request.getOriginalFilename());

  try {
    /* 파일을 저장소에 저장 */
    fileUploadService.storeFileInStorage(fileInfo.getId(), fileInfo.getName(), request.getInputStream());
  } catch (IOException e) {
    e.printStackTrace();
  }

  /* 파일정보를 repository에 저장 */
  fileUploadService.saveFileInfoInRepo(fileInfo);
  return new ResponseEntity<>(fileInfo, HttpStatus.OK);
}
```

### 유닛 테스트

- 흐름

실제 서비스와 다르게 `test.properties`를 설정하여 내부 h2 Database와 로컬 저장소에 파일, 파일 정보를 저장하게 설정

![./file-upload-service-unit-test-flow.png](./file-upload-service-unit-test-flow.png)

- `FileUploadTests.java`

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
/* test.properties 를 config로 읽어옴 */
@TestPropertySource(locations = "classpath:test.properties")
public class FileUploadTests {

  @Autowired
  private WebApplicationContext webApplicationContext;
  private MockMvc mockMvc;

  @Autowired
  private FileInfoRepository fileInfoRepository;

  @Before
  public void runOnceBeforeTestMethod() {
    mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
  }

  @Test
  public void testUploadFile() throws Exception {
    /* json을 object로 변환하기 위해 사용하는 객체 */
    ObjectMapper objectMapper = new ObjectMapper();
    /*
    * 매개변수 설명
    * "file": FileUplaodService의 @RequestParam("file")에서 "file"를 뜻함
    * "filename.txt":  파일 이름,
    * "filecontents":  파일 내용
    */
    MockMultipartFile nomalFile = new MockMultipartFile("file", "filename.txt", MediaType.TEXT_PLAIN_VALUE, "filecontents");

    MockHttpServletRequestBuilder builder = MockMvcRequestBuilders.fileUpload("http://127.0.0.1:123/").file(nomalFile);

    /*
    * andDo(print()): 파일 전송에 대한 디버깅 함수
    * andExpect(status.isOk()): 파일 전송후 응답 200 OK 확인
    */
    MvcResult mvcResult = mockMvc.perform(builder).andDo(print()).andExpect(status().isOk()).andReturn();
    /* 응답 내용 String으로 변환 */
    String jsonResponse = mvcResult.getResponse().getContentAsString();
    /* String 형식으로된 json을 FileInfo 객체로 변환 */
    FileInfo fileInfo = objectMapper.readValue(jsonResponse, FileInfo.class);

    /* repository 적재 확인 */
    assertNotNull(fileInfoRepository.findOne(fileInfo.getId()));
  }
}
```

- `test.properties`

```
server.port=123
# 파일 정보 저장 db 를 내부 h2 이용
spring.datasource.url=jdbc:h2:~/test;AUTO_SERVER=TRUE
# 로컬 저장소 파일 업로드 저장 경로
storage=storage/
```

- `build.gradle`

```groovy
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    ...
    /* h2 database를 사용하기 위해 추가*/
    testCompile('com.h2database:h2')
}
```
