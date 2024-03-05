# 미니프로젝트 1단계  

## 프로젝트 설정
```Gradle
dependencies {
    // Data JPA
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	
	// WEB
	implementation 'org.springframework.boot:spring-boot-starter-web'
	
	// Lombok
	implementation 'org.projectlombok:lombok'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
	
	// MySQL
	runtimeOnly 'com.mysql:mysql-connector-j'
	
	// Test
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	
	// MockMvc
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
	
	// Valid
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	
	//QueryDSL
	implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
	annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
	
	//QClass
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"

}
```

## 구현 기능
1. 팀 등록 기능
2. 직원 등록 기능
3. 팀 조회 기능
4. 직원 조회 기능  

## 팀 기능 구현  
### 팀 등록
회사에 있는 팀을 등록할 수 있습니다.  
팀 이름은 필수 값입니다.
```http request
POST http://localhost:8080/api/v1/team
Content-Type: application/json

{
  "name": "팀명"
}
```  

### 팀 조회
팀 조회기능은 아래 API를 통해서 조회가 가능합니다.
```http request
GET http://localhost:8080/api/v1/team/list
```

### DTO
```Java
@Getter
public class EmployeeCreateRequest {
    private String name;
    private ROLE role;
    private LocalDate joinDate;
    private LocalDate birthday;
    private String teamName;

    public EmployeeCreateRequest() {
    }

    @Builder
    private EmployeeCreateRequest(final String name, final ROLE role, final LocalDate joinDate, final LocalDate birthday, String teamName) {
        this.name = name;
        this.role = role;
        this.joinDate = joinDate;
        this.birthday = birthday;
        this.teamName = teamName;
    }

    public boolean hasTeam() {
        if (teamName == null || teamName.isBlank()) {
            return false;
        }
        return true;
    }

}
```

### Controller
```Java
@RestController
@RequestMapping("/api/v1")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping("/employee/list")
    public ApiResponse<List<EmployeeProfileResponse>> getEmployeeProfileAll() {
        return ApiResponse.ok(employeeService.getEmployeeProfileAll());
    }

    @PostMapping("/employee")
    public ApiResponse<Long> joinEmployee(@RequestBody EmployeeCreateRequest request) {
        return ApiResponse.ok(employeeService.joinEmployee(request));
    }
}
```  
### Service Dto
```Java
@Getter
@NoArgsConstructor
@ToString
public class TeamProfileResponse {
    private String name;
    private String manager;
    private Long memberCount;

    public TeamProfileResponse(final String name, final String manager, final Long memberCount) {
        this.name = name;
        this.manager = manager;
        this.memberCount = memberCount;
    }

}
``` 
### Service
```Java
@Service
public class EmployeeService {
    private final EmployeeRepository employeeRepository;
    private final TeamRepository teamRepository;

    EmployeeService(EmployeeRepository employeeRepository, TeamRepository teamRepository) {
        this.employeeRepository = employeeRepository;
        this.teamRepository = teamRepository;
    }

    public long joinEmployee(EmployeeCreateRequest request) {
        Employee employee = Employee.create(request.getName(), request.getRole(), request.getJoinDate(), request.getBirthday());
        ifHasTeamThenJoinTeam(request, employee);
        return employeeRepository.save(employee);
    }

    private void ifHasTeamThenJoinTeam(EmployeeCreateRequest request, Employee employee) {
        if (request.hasTeam()) {
            Team team = teamRepository.findByName(request.getTeamName())
                    .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 팀명입니다."));
            employee.joinTeam(team.getId());
        }
    }

    public List<EmployeeProfileResponse> getEmployeeProfileAll() {
        return employeeRepository.getEmployeeProfileAll();
    }
}
```  
**Team Repository**
```Java
public interface TeamJpaRepository extends JpaRepository<Team, Long>{
    Optional<Team> findByName(String name);

    @Query("select " +
            "new org.example.miniinflearn.api.team.service.response.TeamProfileResponse(" +
            "t.name, " +
            "function('listagg', case when e.role = MANAGER then e.name else null end,','),"+
            "SUM(CASE WHEN e.teamId is not null then 1L ELSE 0L END)) " +
            "from Team as t " +
            "left join Employee as e on t.id = e.teamId GROUP BY t.name")
    List<TeamProfileResponse> getTeamProfile();
}
```  

### TestCode  
**Service**
```Java
@SpringBootTest
public class EmployeeServiceTest {

    @Autowired
    private EmployeeService employeeService;
    @Autowired
    private TeamService teamService;

    @Test
    @DisplayName("직원을 등록할 수 있다.")
    void joinEmployee() {
        // given
        String name = "둘리";
        ROLE role = ROLE.MANAGER;
        LocalDate joinDate = LocalDate.now();
        LocalDate birthday = LocalDate.of(2000, 3, 4);
        EmployeeCreateRequest request = getEmployeeCreateRequest(name, role, joinDate, birthday);

        // when
        long joinEmployeeId = employeeService.joinEmployee(request);

        // then
        Assertions.assertThat(joinEmployeeId).isEqualTo(1L);
    }

    private EmployeeCreateRequest getEmployeeCreateRequest(String name, ROLE role, LocalDate joinDate, LocalDate birthday) {
        return EmployeeCreateRequest.builder()
                .name(name)
                .role(role)
                .joinDate(joinDate)
                .birthday(birthday)
                .build();
    }

    @Test
    @DisplayName("팀원 정보를 모두 조회할 수 있다.")
    void getMemberAll() {
        //given
        teamService.addTeam("팀A");
        teamService.addTeam("팀B");

        employeeService.joinEmployee(getEmployeeCreateRequest("팀A", "매니저-둘리", ROLE.MANAGER, LocalDate.of(1998, 8, 1), LocalDate.of(2024, 1, 1)));
        employeeService.joinEmployee(getEmployeeCreateRequest("팀A", "직원-둘리", ROLE.MEMBER, LocalDate.of(1997, 8, 1), LocalDate.of(2024, 3, 1)));

        //when
        List<EmployeeProfileResponse> responses = employeeService.getEmployeeProfileAll();

        for (EmployeeProfileResponse respons : responses) {
            System.out.println("respons = " + respons);
        }

        //then
        Assertions.assertThat(responses).hasSize(2);
        Assertions.assertThat(responses)
                .extracting("name", "teamName", "role", "birthday", "workStartDate")
                .containsExactlyInAnyOrder(
                        Assertions.tuple("매니저-둘리", "팀A", "MANAGER", "1998-08-01", "2024-01-01"),
                        Assertions.tuple("직원-둘리", "팀A", "MEMBER", "1997-08-01", "2024-03-01")
                );
    }

    private EmployeeCreateRequest getEmployeeCreateRequest(String teamName, String name, ROLE role, LocalDate birthday, LocalDate workStartDate) {
        return EmployeeCreateRequest.builder()
                .teamName(teamName)
                .name(name)
                .role(role)
                .birthday(birthday)
                .joinDate(workStartDate)
                .build();
    }
}
```  
**Controller**
```Java
@WebMvcTest(controllers = TeamController.class)
class TeamControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private TeamService teamService;

    @Test
    @DisplayName("팀 이름은 필수 값이다.")
    void c1() throws Exception {
        //given
        CreateTeamRequest request = new CreateTeamRequest("팀");

        //when
        mockMvc.perform(MockMvcRequestBuilders.post("/api/v1/team")
                        .content(objectMapper.writeValueAsString(request))
                        .contentType(MediaType.APPLICATION_JSON_VALUE))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.jsonPath("$.code").value("200"));
    }


    @Test
    @DisplayName("팀 이름이 공백일 경우 예외가 발생한다.")
    void c2() throws Exception {
        //given
        CreateTeamRequest request = new CreateTeamRequest("");

        //when
        mockMvc.perform(MockMvcRequestBuilders.post("/api/v1/team")
                        .content(objectMapper.writeValueAsString(request))
                        .contentType(MediaType.APPLICATION_JSON_VALUE))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.jsonPath("$.code").value("400"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.body").value("FAIL"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.message").value("팀명은 공백이나 null이 들어올 수 없습니다."));
    }
}
```  
  
## 직원 기능 구현
### 직원 조회
직원 조회 기능은 아래 API를 통해서 조회가 가능합니다.
```http request
GET http://localhost:8080/api/v1/employee/list
```

### 직원 등록
직우너 등록 기능은 아래 API를 통해서 등록이 가능합니다.
```http request
POST http://localhost:8080/api/v1/employee
Content-Type: application/json

{
  "name": "직원이름",
  "teamName": "소속 팀 이름"
  "role": "MANAGER",
  "birthday": "1998-01-01",
  "workStartDate": "2024-01-01",
}
```  
  
**DTO**
```Java
@Getter
public class EmployeeCreateRequest {
    private String name;
    private ROLE role;
    private LocalDate joinDate;
    private LocalDate birthday;
    private String teamName;

    public EmployeeCreateRequest() {
    }

    @Builder
    private EmployeeCreateRequest(final String name, final ROLE role, final LocalDate joinDate, final LocalDate birthday, String teamName) {
        this.name = name;
        this.role = role;
        this.joinDate = joinDate;
        this.birthday = birthday;
        this.teamName = teamName;
    }

    public boolean hasTeam() {
        if (teamName == null || teamName.isBlank()) {
            return false;
        }
        return true;
    }

}
```
**Controller**
```Java
@RestController
@RequestMapping("/api/v1")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping("/employee/list")
    public ApiResponse<List<EmployeeProfileResponse>> getEmployeeProfileAll() {
        return ApiResponse.ok(employeeService.getEmployeeProfileAll());
    }

    @PostMapping("/employee")
    public ApiResponse<Long> joinEmployee(@RequestBody EmployeeCreateRequest request) {
        return ApiResponse.ok(employeeService.joinEmployee(request));
    }
}
```
**Service Dto**
```Java
@Getter
@AllArgsConstructor
@ToString
public class EmployeeProfileResponse {

    private String name;
    private String teamName;
    private String role;
    private String birthday;
    private String workStartDate;

}
```  

**Service**
```Java
@Service
public class EmployeeService {
    private final EmployeeRepository employeeRepository;
    private final TeamRepository teamRepository;

    EmployeeService(EmployeeRepository employeeRepository, TeamRepository teamRepository) {
        this.employeeRepository = employeeRepository;
        this.teamRepository = teamRepository;
    }

    public long joinEmployee(EmployeeCreateRequest request) {
        Employee employee = Employee.create(request.getName(), request.getRole(), request.getJoinDate(), request.getBirthday());
        ifHasTeamThenJoinTeam(request, employee);
        return employeeRepository.save(employee);
    }

    private void ifHasTeamThenJoinTeam(EmployeeCreateRequest request, Employee employee) {
        if (request.hasTeam()) {
            Team team = teamRepository.findByName(request.getTeamName())
                    .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 팀명입니다."));
            employee.joinTeam(team.getId());
        }
    }

    public List<EmployeeProfileResponse> getEmployeeProfileAll() {
        return employeeRepository.getEmployeeProfileAll();
    }
}
```
**Repository**
```Java
public interface EmployeeJpaRepository extends JpaRepository<Employee, Long> {

    @Query("select " +
            "new org.example.miniinflearn.api.employee.service.EmployeeProfileResponse(" +
            "e.name,t.name,str(e.role),cast(e.birthday as String),cast(e.workStartDate as String))" +
            "from Employee as e left join Team as t on e.teamId = t.id")
    List<EmployeeProfileResponse> getEmployeeProfileAll();
}
```  

### Employee Test Code
```Java
@SpringBootTest
public class EmployeeServiceTest {

    @Autowired
    private EmployeeService employeeService;
    @Autowired
    private TeamService teamService;

    @Test
    @DisplayName("직원을 등록할 수 있다.")
    void joinEmployee() {
        // given
        String name = "둘리";
        ROLE role = ROLE.MANAGER;
        LocalDate joinDate = LocalDate.now();
        LocalDate birthday = LocalDate.of(2000, 3, 4);
        EmployeeCreateRequest request = getEmployeeCreateRequest(name, role, joinDate, birthday);

        // when
        long joinEmployeeId = employeeService.joinEmployee(request);

        // then
        Assertions.assertThat(joinEmployeeId).isEqualTo(1L);
    }

    private EmployeeCreateRequest getEmployeeCreateRequest(String name, ROLE role, LocalDate joinDate, LocalDate birthday) {
        return EmployeeCreateRequest.builder()
                .name(name)
                .role(role)
                .joinDate(joinDate)
                .birthday(birthday)
                .build();
    }

    @Test
    @DisplayName("팀원 정보를 모두 조회할 수 있다.")
    void getMemberAll() {
        //given
        teamService.addTeam("팀A");
        teamService.addTeam("팀B");

        employeeService.joinEmployee(getEmployeeCreateRequest("팀A", "매니저-둘리", ROLE.MANAGER, LocalDate.of(1998, 8, 1), LocalDate.of(2024, 1, 1)));
        employeeService.joinEmployee(getEmployeeCreateRequest("팀A", "직원-둘리", ROLE.MEMBER, LocalDate.of(1997, 8, 1), LocalDate.of(2024, 3, 1)));

        //when
        List<EmployeeProfileResponse> responses = employeeService.getEmployeeProfileAll();

        for (EmployeeProfileResponse respons : responses) {
            System.out.println("respons = " + respons);
        }

        //then
        Assertions.assertThat(responses).hasSize(2);
        Assertions.assertThat(responses)
                .extracting("name", "teamName", "role", "birthday", "workStartDate")
                .containsExactlyInAnyOrder(
                        Assertions.tuple("매니저-둘리", "팀A", "MANAGER", "1998-08-01", "2024-01-01"),
                        Assertions.tuple("직원-둘리", "팀A", "MEMBER", "1997-08-01", "2024-03-01")
                );
    }

    private EmployeeCreateRequest getEmployeeCreateRequest(String teamName, String name, ROLE role, LocalDate birthday, LocalDate workStartDate) {
        return EmployeeCreateRequest.builder()
                .teamName(teamName)
                .name(name)
                .role(role)
                .birthday(birthday)
                .joinDate(workStartDate)
                .build();
    }
}
```