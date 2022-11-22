## Mapstruct 이란 ? 
- Object Mapping 라이브러리 중 하나
- 자바에서 객체 간 매핑에 대한 코드를 자동으로 생성해주는 매핑 라이브러리
- 장점
    1. 컴파일 시 오류를 확인할 수 있다.
    2. 리플렉션(Reflction)을 사용하지 않기 때문에 매핑 속도가 빠르다 !!!
        - 많이 사용하는 Model Mapper의 경우 런타임 시점에 reflection으로 모델 매핑을 하게 됨.  
        
        - 런타임 시 reflection 자주 이루어진다면 앱 성능이 좋지 않게된다.   
        
        - MapStruct의 경우 컴파일 시점에 매핑 클래스를 생성하고 그 구현체를 런타임에 사용함.  
        - 참고 : [리플렉션 특징](https://kdg-is.tistory.com/entry/JAVA-%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98-Reflection%EC%9D%B4%EB%9E%80)
        
    3. 디버깅이 쉽다.
    4. 생성된 매핑 코드를 눈으로 직접 확인할 수 있다.

- 성능 비교 [Performance of Java Mapping Frameworks | Baeldung](https://www.baeldung.com/java-performance-mapping-frameworks)
   

## 사용 방법

### 기본적인 매핑 방법
- 자바 인터페이스에 매핑 메소드 정의한다.
- `org.mapstruct.Mapper` @Mapper 어노테이션 붙이기 
```java
@Mapper
public interface MessageMapper {
    MessageEntity toMessageEntity(DTO dto);
}
```
- @Mapper 가 붙은 인터페이스는 MapStruct Code Generator가 해당 인터페이스의 구현체를 생성해준다.

### 서로 다른 속성 매핑 
- ex) `StudyFeedback` 엔티티의 receivedUser 객체의 id 속성을 FeedBackResponseDTO 의 receivedUserId 로 매핑시켜야 한다.

StudyFeedBack Entity
```java
@Entity @Getter
@NoArgsConstructor
@Table(name = "tbl_study_feedback")
public class StudyFeedback extends CreatedBaseEntity {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "received_user_id", nullable = false)
    private User receivedUser; // 받은 사람 (조회할 때 사용할 필드)

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "send_user_id", nullable = false)
    private User sendUser; // 보낸 사람

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "study_history_id", nullable = false)
    private StudyHistory studyHistory;

    @NotNull
    private Byte score;

    @NotNull
    private Boolean passOrFail;
}
```

FeedBackResponseDTO
```java
@Getter @Setter
public static class FeedBackResponseDTO {
    private Long id;
    private String receivedUserId;
    private Byte score;
    private Boolean passOrFail;
}
```

- @Mapping 에 소스와 타깃 소스 속성을 설정하면 된다.
```java
@Mapper(componentModel = "spring", unmappedSourcePolicy = ReportingPolicy.IGNORE)
public interface GroupStudyMapper {
    @Mapping(source = "receivedUser.id", target = "receivedUserId")
    GroupStudyDTO.FeedBackResponseDTO toFeedBackResponseDto(StudyFeedback studyFeedback);
}
```

### Collection Mapping
- MapStruct가 자동으로 생성하는 매핑 코드를 불가피하게 쓰지 못하는 경우 Mapper에 커스텀 메소드를 작성하여 사용 

- 문제 
  - ex) 
  ```java
    @Mapping(source = "questionList.id", target = "questionListId")
    SelfHistoryDTO.SelfHistoryResponseDTO[] toResponseArray(List<SelfHistory> selfHistoryList);
  ```
  - 위와 같은 경우 `SelfHistory`안의 questionList 객체의 값을 dto의 값들과 매핑시켜주어야 했음
  - 그러나 단일 객체가 아니라 컬렉션 객체 매핑의 경우 @Mapping 어노테이션이 내부 매핑에 적용되지 않음!   

- 방법 
1. 인터페이스에 default 메소드로 커스텀 매핑 추가 
  ```java
  default SelfHistoryResponseDTO toResponseArray(SelfHistory selfHistory) {
        SelfHistoryResponseDTO selfHistoryResponseDTO = new SelfHistoryResponseDTO();

        selfHistoryResponseDTO.setId(selfHistory.getId());
        selfHistoryResponseDTO.setQuestionListId(selfHistory.getQuestionList().getId());
        selfHistoryResponseDTO.setQuestionListEnterprise(selfHistory.getQuestionList().getEnterprise());
        selfHistoryResponseDTO.setQuestionListJob(selfHistory.getQuestionList().getJob());
        selfHistoryResponseDTO.setSavedLocation(selfHistory.getSavedLocation());
        selfHistoryResponseDTO.setCreatedAt(selfHistory.getCreatedAt());

        return selfHistoryResponseDTO;
    }
  ```
   

2. Mapper를 추상 클래스로 정의하는 방법
```java
@Mapper 
public abstract class SelfHistoryMapper { 
    @Mapping(...) 
    ... 
    
    public abstract SelfHistoryDTO.SelfHistoryResponseDTO[] toResponseArray(List<SelfHistory> selfHistoryList);

    public SelfHistoryResponseDTO toResponseArray(SelfHistory selfHistory) {
        SelfHistoryResponseDTO selfHistoryResponseDTO = new SelfHistoryResponseDTO();

        selfHistoryResponseDTO.setId(selfHistory.getId());
        selfHistoryResponseDTO.setQuestionListId(selfHistory.getQuestionList().getId());
        selfHistoryResponseDTO.setQuestionListEnterprise(selfHistory.getQuestionList().getEnterprise());
        selfHistoryResponseDTO.setQuestionListJob(selfHistory.getQuestionList().getJob());
        selfHistoryResponseDTO.setSavedLocation(selfHistory.getSavedLocation());
        selfHistoryResponseDTO.setCreatedAt(selfHistory.getCreatedAt());

        return selfHistoryResponseDTO;
  }
```
- 참고 : https://www.baeldung.com/java-mapstruct-mapping-collections

