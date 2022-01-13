## 어노테이션

- @RequestParam : QueryString 받을 때 이용 (기본 데이터 타입은 생략 가능 (String도 포함))
- @PathVariable : 경로변수 받을 때 이용
- @RequestBody : HTTP message body 데이터를 받을 때 이용 (String, 객체 모두 가능)
- @ResponseBody : HTTP message body에 데이터를 넣어 응답 (객체 가능)
- @ResponseStatus : HTTP Response Status Code 설정 (동적으로 변경하는 것은 불가능하므로, 동적으로 변경하려면 ResponseEntity를 반환형으로 이용해야한다.)

```java
@PostMapping("/save")
public String save(
    @RequestParam("username") String username,
    @RequestParam("age") int age,
    Model model) {
    Member member = new Member(username, age);
    memberRepository.save(member);
    model.addAttribute("member", member);
    return "save-result";
}

@ResponseBody
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}

@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
    Item item = itemRepository.findById(itemId);
    model.addAttribute("item", item);
    return "basic/editForm";
}

@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
    log.info("username={}, age={}", data.getUsername(), data.getAge());
    return "ok";
}

@ResponseStatus(HttpStatus.OK)
@ResponseBody
@GetMapping("/response-body-json-v2")
public HelloData responseBodyJsonV2() {
    HelloData helloData = new HelloData();
    helloData.setUsername("userA");
    helloData.setAge(20);
    return helloData;
}

@GetMapping("/response-body-string-v2")
public ResponseEntity<String> responseBodyV2() {
    return new ResponseEntity<>("ok", HttpStatus.OK);
}

@GetMapping("/response-body-json-v1")
public ResponseEntity<HelloData> responseBodyJsonV1() {
    HelloData helloData = new HelloData();
    helloData.setUsername("userA");
    helloData.setAge(20);
    return new ResponseEntity<>(helloData, HttpStatus.OK);
}
```

## API 설계

### Entity를 바로 사용하지 않도록 하자

- Controller 메서드에서 Entity를 파라미터로 바로 받지 않도록 하자
  - 파라미터로 Entity를 바로 받게 되면 Entity의 필드가 변경되었을 때 API 스펙 자체가 변해버리게 된다
  - 따라서 파라미터를 받는 전용 DTO를 만들어서 사용하자

```java
// Member: Entity, CreateMemberReqeust: DTO

@PostMapping("/api/v1/members")
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}

@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}

@Data
static class CreateMemberRequest {
    private String name;
}

@Data
static class CreateMemberResponse {
    private Long id;

    public CreateMemberResponse(Long id) {
        this.id = id;
    }
}
```

- 마찬가지로 Controller 메서드에서 Entity를 그대로 반환하지 않도록 하자
  - API를 호출하는 데마다 원하는 정보가 다를 수 있다
  - Entity에 Presentation layer의 요구사항이 들어가게 되면 다양한 응답을 만들 수 없다
  - 따라서 응답 전용 DTO를 만들어서 반환하자
- Controller 메서드에서 Array(Collection) 자체를 반환하지 않도록 하자
  - Collection을 반환하는 경우는 다음과 같이 한번 감싸서 반환하도록 하자

```java
@GetMapping("/api/v2/members")
public Result membersV2() {
    List<Member> findMembers = memberService.findMembers();
    //엔티티 -> DTO 변환
    List<MemberDto> collect = findMembers.stream()
            .map(m -> new MemberDto(m.getName()))
            .collect(Collectors.toList());

    return new Result(collect);
}

@Data
@AllArgsConstructor
static class Result<List<MemberDto>> {
    private List<MemberDto> data;
}

@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}

```
