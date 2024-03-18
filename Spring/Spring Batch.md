# Spring Batch란?
대용량 데이터를 처리하기 위한 프레임워크
대량의 데이터를 처리하거나 주기적이고 반복적인 작업을 실행하는데 사용
## vs Scheduler
- 배치는 일괄적으로 대량의 데이터를 처리하는 것을 의미하며 스케줄러와 함께 쓰일 뿐
- 스케줄러의 경우 주어진 작업을 미리 정의된 시간에 실행할 수 있게 해주는 소프트웨어
## 구성 요소
### Job
### 1️⃣ Job
- 배치 처리의 전체 프로세스
- 하나 또는 그 이상의 Step을 포함하며 스프링 배치 계층에서 가장 상위
- 각 Job은 고유한 이름을 가지며, 이 이름은 JobInstance를 구별하는데 사용
### 2️⃣ JobInstance
- 특정 Job의 실제 실행 인스턴스
	> 매일 12시에 데이터를 처리하는 Job일 경우, 매일 매일 새로운 JobInstance가 생성
- 한 번 생성된 JobInstance는 해당 날짜의 데이터를 처리하는 데 사용되며, 실패했을 경우 같은 JobInstance를 다시 실행하여 작업 시도
### 3️⃣ JobParameters
- JobInstance를 생성하고 구별하는데 사용되는 파라미터
- Job이 실행될 때 필요한 파라미터이자 JobInstance를 구별
- String, Double, Long, Date 타입의 파라미터 지원
### 4️⃣ JobExecution
- JobInstance의 한 번의 시행 시도
- 같은 JobInstance를 재시도해도 새로운 JobExcution 생성
- 실행 상태, 시작 시간, 종료 시간, 생성 시간 등 JobInstance의 실행에 대한 세부 정보를 가짐
### 5️⃣ JobRepository
- 배치 작업에 관련된 모든 정보를 저장하고 관리
- JobExecutionContext, StepExecutionContext, JobParameters를 저장하고 관리
- Job이 실행될 때 새로운 JobExecution, StepExecution을 생성하고 실행 상태 추적 -> Job의 실행 도중인 상태
### 6️⃣ JobLauncher
- Job과 JobParameters를 받아 Job을 실행하는 역할
- Job의 생명 주기를 관리하며, JobRepository를 통해 실행 상태 유지
### 7️⃣ JobOperator
- 외부 인터페이스
- Job의 실행, 중지, 재시작 등의 흐름제어
- JobLauncher와 JobRepository에 대한 직접적인 접근 없이도 배치 작업을 수행하고 상태 조회
### 8️⃣ JobExplorer
- Job의 실행 이력을 조회
- JobRepository와 유사하지만 읽기 전용 접근에 초점
### Step
### 1️⃣ Step
- Job의 하위 단계
- 실제 배치 처리 작업이 이루어지는 단위
- 한 개 이상의 Step이 Job으로 구성되며 각 Step은 순차적으로 처리
- 각 Step은 Chunk 방식 혹은 Tasklet 방식으로 구성
### 2️⃣ StepExecution
- Step의 한 번의 실행
- Step의 실행 상태, 실행 시간 등의 정보 포함
- 각 Step의 시도마다 새로운 StepExcution 생성
### Execution
### 7️⃣ ExecutionContext
- Step간 또는 Job 실행 도중 데이터를 공유하는데 사용되는 저장소
- JobExecutionContext, StepExecutionContext가 존재
- Job 혹은 Step이 실패했을 경우, ExecutionContext를 통해 마지막 실행 상태를 재구성하여 재시도 또는 복구 작업 수행
### Item
### 1️⃣ ItemReader
- 배치 작업에서 처리할 아이템을 읽음
### 2️⃣ ItemProcessor
- ItemReader로부터 읽어온 아이템을 처리하는 역할
- 데이터 필터링, 변환 등의 작업
### 3️⃣ ItemWriter
- ItemProcessor에서 처리된 데이터를 최종적으로 기록
- 다양한 형태의 구현체를 통해 데이터베이스에 기록하거나, 파일을 생성하거나, 메시지를 발행하는 등의 형식으로 데이터를 씀
### 예시
```java
@Component
public class BatchJobLauncher implements CommandLineRunner {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job; // 자동 주입을 위해서는 Job 빈이 정의되어 있어야 합니다.

    @Override
    public void run(String... args) throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                .addLong("time", System.currentTimeMillis()) // 실행마다 고유하게 파라미터를 생성
                .toJobParameters();
        JobExecution execution = jobLauncher.run(job, jobParameters);

        System.out.println("Job Status : " + execution.getStatus());
    }
}
```
- **Job**: "DailyOrderReportJob"
    - 목적 : 매일 주문 데이터를 집계하여 리포트를 생성
	```java
	@Bean 
	public Job dailyOrderReportJob(JobCompletionNotificationListener listener, 
				Step readOrders, Step generateReport) { 
		return jobBuilderFactory.get("dailyOrderReportJob") 
			.incrementer(new RunIdIncrementer())
			.listener(listener) 
			.start(readOrders()) 
			.next(generateReport()) 
			.build(); 
	}
	```
- **Step 1**: "readOrders"
    - **ItemReader**: 데이터베이스에서 그날의 주문 데이터(`Item`)를 읽음
    - **ItemProcessor**: (이 경우에는 생략될 수 있음, 단순히 주문 데이터를 집계만 하는 경우)
    - **ItemWriter**: 읽어온 주문 데이터를 중간 집계 형태로 임시 저장
	```java
	// Step 1: "readOrders" - 데이터베이스에서 주문 데이터를 읽어오는 ItemReader
	public ItemReader<Order> orderItemReader() {
		return new JdbcCursorItemReaderBuilder<Order>()
				.dataSource(dataSource)
				.name("orderItemReader")
				.sql("SELECT id, date, amount FROM orders WHERE date = CURRENT_DATE")
				.rowMapper(new BeanPropertyRowMapper<>(Order.class))
				.build();
	}

	// Step 1의 ItemWriter - 중간 집계 데이터를 임시 저장
	public ItemWriter<Order> orderItemWriter() {
		return new JdbcBatchItemWriterBuilder<Order>()
				.dataSource(dataSource)
				.sql("INSERT INTO temp_aggregate (id, summary) VALUES (:id, :summary)")
				.beanMapped()
				.build();
	}
	
	// Step 1: "readOrders" 스텝 정의
	@Bean
	public Step readOrders() {
		return stepBuilderFactory.get("readOrders")
				.<Order, Order> chunk(10)
				.reader(orderItemReader())
				.writer(orderItemWriter())
				.build();
	}
	```
- **Step 2**: "generateReport"
    - **ItemReader**: Step 1에서 생성된 중간 집계 데이터를 읽음
    - **ItemProcessor**: 주문 데이터를 분석하고 리포트 형식으로 가공
    - **ItemWriter**: 최종 가공된 리포트를 PDF 파일 등의 형식으로 저장하거나, 이메일로 발송
	```java
    // Step 2: "generateReport" - 중간 집계 데이터를 읽어오는 ItemReader
    public ItemReader<AggregateData> aggregateDataReader() {
        return new JdbcCursorItemReaderBuilder<AggregateData>()
                .dataSource(dataSource)
                .name("aggregateDataReader")
                .sql("SELECT summary FROM temp_aggregate")
                .rowMapper(new BeanPropertyRowMapper<>(AggregateData.class))
                .build();
    }

    // Step 2의 ItemWriter - 최종 리포트를 생성
    public ItemWriter<Report> reportItemWriter() {
        // PDF 생성, 이메일 발송 등의 로직 구현
    }

    // Step 2: "generateReport" 스텝 정의
    @Bean
    public Step generateReport() {
        return stepBuilderFactory.get("generateReport")
                .<AggregateData, Report> chunk(10)
                .reader(aggregateDataReader())
                //.processor(reportItemProcessor()) // 필요한 경우
                .writer(reportItemWriter())
                .build();
    }
	```
## Step 처리 방식
### 1️⃣ Chunk
- 대용량 데이터 처리에 사용
- 큰 데이터를 일련의 작은 묶음 (Chunk)로 나누고, 각 Chunk를 개별적인 트랜잭션 범위 내에서 처리하는 방식
- Chunk는 트랜잭션 범위로, Chunk Size는 한 번에 처리될 데이터 항목의 수를 의미
- Paging Size가 5이고 Chunk Size가 10인 경우, 2번의 Read가 이루어진 후에 1번의 Transaction 수행하므로 비효율적
> 💡 페이지 크기와 동일한 Chunk Size를 사용하는 것이 좋음
![](https://i.imgur.com/dmuOpK5.png)
- **Reader**
	- 데이터 소스로부터 데이터를 읽어와 Chunk 생성
	- 데이터는 일반적으로 db, file, message queue 등이 될 수 있음
- **Processor**
	- 읽어온 데이터에 대해 필요한 처리
	- 데이터 검증, 필터링, 변환 등
- **Writer**
	- 처리된 데이터를 최종적으로 저장

```java
@Bean
public Step sampleStep() {
    return stepBuilderFactory.get("sampleStep")
            .<InputType, OutputType>chunk(10)
            .reader(myItemReader())
            .processor(myItemProcessor())
            .writer(myItemWriter())
            .build();
}
```
### 2️⃣Tasklet 방식
![](https://i.imgur.com/iMZ0M7l.png)
- 더 단순하거나 반복적이지 않은 일회성 작업을 정의할 때 사용
- 기본적으로 하나의 작업을 수행하는 방식
- `execute()` 메서드를 구현하여 사용하며 하나의 트랜잭션 범위에서 실행
- `execute()` 메서드는 `RepeatStatus`를 반환하며 이는 Tasklet의 실행 상태를 나타냄
	- `RepeatStatus.FINISHED` : Tasklet의 처리가 완료됨
	- `RepeatStatus.CONTINUABLE` : Tasklet이 계속 실행되어야 함

```java
@Bean
public Step sampleTaskletStep() {
    return stepBuilderFactory.get("sampleTaskletStep")
            .tasklet(new Tasklet() {
                @Override
                public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) 
		                throws Exception {
					// business logic
                    return RepeatStatus.FINISHED;
                }
            })
            .build();
}
```
