# Airflow-study

![untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/edefa2d6-55b8-489a-9c11-831aa5476cde/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T140230Z&X-Amz-Expires=86400&X-Amz-Signature=a13543f91a455cfc404a696748ecaede52211828421b7f8bdc0285181ab09701&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

## 💡mastering DAGs

### execution_date 의 이해

> ***start_date** — date at which DAG will start being scheduled

**schedule_interval** — the interval of time from the minimum start_date at which we want our DAG to be triggered.*

> 

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9a8d7efa-5c6f-4393-a5f4-c19eb6b4af6a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132913Z&X-Amz-Expires=86400&X-Amz-Signature=8a6844ea7ed6c7315550c0f88bd46d7df5124f8c1e7fcdcd88d034a9f0095523&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

> **start_date보다 늦게 DAG가 실행되는 이유**
>
> airflow의 컨셉이라고 생각하면 될 것 같다. 예를 들어, 매일 하루치의 데이터를 쌓아서 처리하는 서비스가 있다고 가정했을 때  start_date(5월 5일 00시)는 서비스가 시작된 날이다. 이후 24시간 동안 쌓인 데이터를 바탕으로 실제로 데이터 처리 작업이 처음 발생한 시간(첫 번째 `DagRun` )은 5월 6일 00시가 된다.

```python
DAG = {
...
'start_date': datetime(2022, 5, 5),
schedule_interval="@daily",
...
}
```

- DAG는 `2022-05-06`에 최초 실행되며, `execution_date`는 `2022-05-05`

  첫 번째 `DagRun`의 trigger는 5월 5일 이후 자정 = 5월 6일 00시가 된다. 첫 번째 `DagRun`이 실행 가능한 time range는 5월 6일 00 ~ 5월 7일 00시 (24hour, `@daily`)

- `2022-05-07` 에 실행된 DagRun의 `execution_date`는 `2022-05-06`,

- `2022-05-08` 에 실행된 DagRun의 `execution_date`는 `2022-05-07`,

  즉, execution_date는 실제 작업의 runtime과는 무관한 해당 작업을 나타내는 **id값**이라고 생각하면 된다.

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4de9aa96-9029-4dcb-9389-3a5764350ccf/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132932Z&X-Amz-Expires=86400&X-Amz-Signature=21b37ba50695cf0426d8d244c70ed3cb4c1bcafdfd2f7409019657ae38601c0c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

```python
some_bash_cmd = f"cd {scripts_folder}" + " && ./start.sh some_daily '{{ yesterday_ds }}'"
```

참고로 스케줄러를 통해 실행되는 DAG의 경우 execution_date가 자동으로 할당된다.

CLI를 통해 명령어로 실행하는 경우 execution_date를 반드시 명시해 주어야 한다.

```python
DAG = {
...
'start_date': datetime(2022, 5, 5),
schedule_interval="@hourly",
...
}
```

- execution_date는 실제 작업의 runtime과는 무관

| JOB No | DAG id  | execution_date    | start_date(runtime) | end_date          |
| ------ | ------- | ----------------- | ------------------- | ----------------- |
| 1      | zum_dag | 22-05-05T00:00:00 | 22-05-05T01:00:47   | 22-05-05T01:01:62 |
| 2      | zum_dag | 22-05-05T01:00:00 | 22-05-05T02:06:01   | 22-05-05T02:07:40 |
| 3      | zum_dag | 22-05-05T02:00:00 | 22-05-05T03:03:70   | 22-05-05T03:04:01 |

- 참고로 코드에서 DAG를 선언할 때의 start_date값은 DAG의 선언 시간을 뜻하지만( 여기선 2022-05-05T00:00이 된다.), airflow scheduler UI의 start_date는 실제 job이 동작한 시간을 말한다.

스케줄링된 daily 배치 실행 만이 목적이라면 execution_date을 사용하지 않고, 코드 내에서 date_time.now()를 이용가능

하지만 동일한 배치의 adhoc 한 단발성 실행 시에도 (i.e. backfill) 동일한 dag 코드를 활용하려면 execution_date을 사용할 수밖에 없다.

따라서 전날의 자료 또는 이전 실행 자료를 이용해 ETL을 하고 싶은 경우에는 yesterday_ds, prev_ds를 사용해야 한다.

(하루 전날의 로그 데이터를 통해 ETL하는 작업이 필요하다면, `yesterday_ds` 대신 `ds`를 사용)

### (참고) timezone 설정

airflow는 기본적으로 **UTC timezone**을 사용함. 따라서 korea timezone으로 나타내기 위해서는 시간 설정 변경이 필요함.( 참고 : Asia/Seoul은 UTC+09, 우리나라 시간 대비 9시간 전 )

- **DAG에서 object를 활용**

  airflow에서 TIMEZONE 변환을 위해 **pendulum** 패키지를 사용할 수 있다.

  ```python
  import pendulum
  from datetime import datetime
  
  local_tz = pendulum.timezone("Asia/Seoul")
  
  default_args=dict(
      start_date = datetime(2020, 2, 28, 2, tzinfo=local_tz)
  ```

- **환경 변수 설정**

  ```python
  # airflow.cfg
  ...
  #default_timezone = utc
  default_timezone = Asia/Seoul   <- Pendulum 라이브러리에서 지원하는 timezone 형태
  ```

- **다양한 execution_date 파라미터**

  ```python
  def _print_context(**context):
  		start=context["ececution_date"]
  		end=context["next_execution_date"]
  		print(f"Start: {start}, end: {end}")
  
  print_context=PythonOperator(
  		task_id="print_context", python_callable=_print_context, dag=dag
  )
  
  ## 출력 예:
  ## Start: 2022-05-01T14:00:00+00:00, end 2022-05-01T15:00:00+00:00
  ```

  - airflow 에서 제공하는 모든 task context variable 및 jinja template

    https://airflow.apache.org/docs/apache-airflow/stable/templates-ref.html

  ```python
  ## Airflow 1.x   -> 콘텍스트 변수 사용을 위해서는 옵션을 설정해줘야함.
  PythonOperator(
  		task_id="pass_context",
  		python_callable=_pass_context,
  		provide_context=True,
  		dag=dag
  		...
  ## Airflow 2.x   -> 콘텍스트 변수에 대한 호출 여부를 자동으로 판단함.
  PythonOperator(
  		task_id="pass_context",
  		python_callable=_pass_context,
  		dag=dag
  		...
  ```

### backfill and catchup

- Backfill

  과거부터 현재까지 실행되었어야 하는 DAG를 다시 실행하는 과정

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2fa75d3f-db35-4e9e-8ab3-af028d262c21/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132951Z&X-Amz-Expires=86400&X-Amz-Signature=b09deab3e9122acd20dec644f657eae8ab86c3893cedad0ddec5bc4f4aa4ab18&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/11826649-1bb7-4bdd-9050-07223a9580a3/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132959Z&X-Amz-Expires=86400&X-Amz-Signature=677424b5e216c57f3933f28bb344b9bb5ab7b1d0525ecfe4fcfc1611ec258658&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

CLI를 통해 backfill을 다시할 경우

```bash
airflow backfill -s 2020-05-05 -e 2020-05-10 <dag_id>
```

backfill은 기본적으로 실행되지 않은 Task들만 실행하기 때문에 DAG을 아예 다시 실행하고 싶다면 `-reset_dagruns`를 같이 인자로 줘야 함

```bash
airflow backfill -s 2020-01-05 -e 2020-01-10 --reset_dagruns <dag_id>
```

**실패한 작업에 한해** backfill을 실행해야 하는 경우 다음처럼 사용할 수 있음. (catchup 파라미터와 무관하게 동작)

```bash
airflow backfill -s 2020-01-05 -e 2020-01-10 --rerun_failed_tasks <dag_id>
```

- **Catch-up**

catchup 파라미터를 True로 설정하여 backfill 작업이 실행되게 할 수 있다.

```python
with DAG(
		dag_id='my_dag',
		catchup=True,
) as dag:
```

### depends_on_past, wait_for_downstream

task의 성공 여부에 따라 ETL pipeline에 조건을 주고 싶은 경우에 사용하는 옵션 ( 이전 task와 상관없이 실행 가능한 작업에 경우는 사용하지 않음. `default : False`)

- **dpends_on_past**

  이전 날짜의 task 인스턴스 중에서 동일한 task 인스턴스가 실패한 경우 실행되지 않고 대기.

  ```jsx
  default_args = {
      'start_date': datetime(2021, 5, 1),
      "retries": 0,
      "depends_on_past": True
  }
  ```

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e6f688d8-405e-4f1f-affb-3540541557c5/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T133012Z&X-Amz-Expires=86400&X-Amz-Signature=91e54fbe8a0b253160426de95d3aa130b8154352d6c00ee3e1691c3d8d900b0f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

- **wait_for_downstream**

  이전 날짜의 task 인스턴스 중 하나라도 실패한 경우에는 해당 DAG는 실행되지 않고 대기.

  ```jsx
  default_args = {
      'start_date': datetime(2021, 5, 1),
      "retries": 0,
      "wait_for_downstream": True
  }
  ```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/efa2711a-0f16-45a3-905e-6fef40883d65/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T133026Z&X-Amz-Expires=86400&X-Amz-Signature=586eb5a6e1389ec8987a5d1edd35988b53d2b9de4a57360c6b4b28ae22b6d5ff&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

- 이전 DAGRUN의 작업이 완료되기 전에 다음 DAGRUN이 실행되지 않음. ( 다음 DAGRUN이 이전 DAGRUN의 결과를 참조하는 경우 유용함.)

### Retry, Alerting

- dagrun_timeout : DAG가 실행되는데 걸리는 최대 시간, timeout이 지나면 새로운DagRun을 생성

```python
def dag_success_alert(context):
		print(f"task succeed - {context['runid']}")

def dag_failure_alert(context):
		print(f"task failed - {context['runid']}")

def dag_retry_alert(context):
		print(f"dag retried -{context['runid']}")

with DAG(
		dag_id='alert_dag',
		schedule_interval="0 0 * * *",
		start_date=datetime(2021, 1, 1),
		catchup=False,
		dagrun_timeout=timedelta(seconds=75),
		retry_delay=timedelta.second(60), 
		retries=3,
		on_retry_callback=dag_retry_alert
		on_success_callback=None,
		on_failure_callback=dag_failure_alert, ## DAG 단계에서의 실행 옵션
) as dag:
	
	task1 = DummyOperator(task_id="task1")
  task2 = DummyOperator(task_id="task2")
  task3 = DummyOperator(task_id="task3", on_success_callback=dag_success_alert)
																					 ## task 단계에서의 실행 옵션
  task1 >> task2 >> task3
```

- 참고) 오늘의 집 slack notifications

  https://gist.github.com/ddelange/6e33f8f0df3a97d4a371d055aa2d58ac

- max_active_runs : 동시에 실행될 수 있는 DAGrun의 수

- concurrency : 동시에 실행될 수 있는 task의 수

- task_concurrency : 동시에 실행될 수 있는 동일한 task의 수

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9b20188b-2c10-4def-b06d-e74a129af3fc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T133037Z&X-Amz-Expires=86400&X-Amz-Signature=b1291b6f22b5ef2e2cecc3d4c88f4d8530308977bd37b113ed91dfed4eb8cb81&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### Xcom

Airflow는 기본적으로 하나의 task의 결과가 다른 task에 영향을 주지 않는다. 각각이 독립적으로 실행되기 때문에 서로 통신할 수단이 없다. 하지만 workflow를 만들다 보면 이전 작업의 결과, 요소 등을 다음 작업에 전달할 경우가 생긴다. 이때 Xcom을 이용해 전달할 수 있다.

Xcom을 이용해 데이터를 전달하는 경우 DataFrame이나 많은 양의 데이터를 전달하는 것은 지원하지 않으며, 소량의 데이터만 전달하는 것을 권장한다. (MAX XCOM size 48Kb)

Xcom을 사용하기 위해서는 각 task에서 push, pull 하는 방식으로 기본적으로 사용되지만, PythtonOperator의 경우 return이 자동적으로 Xcom 변수로 지정되게 된다.

- 공식문서

  https://airflow.apache.org/docs/apache-airflow/stable/concepts/xcoms.html?highlight=xcom

X**com 사용방법**

- PythonOperator return 값을 이용한 Xcom 사용
- push-pull을 이용한 Xcom 사용
- Jinja templates을 이용한 Xcom 사용

1. **PythonOperator 이용**

   PythonOperator를 이용하면 return 값이 Xcom으로 자동 push됨.

   ```python
   def return_xcom():
       return "xcom!"
       
   return_xcom = PythonOperator(
       task_id = 'return_xcom',
       python_callable = return_xcom,
       dag = dag
   )
   ```

2. **push-pull을 이용**

   `context['task_instance']`를 이용하여 xcom에 push, pull 하여 데이터를 주고받는 방법

   - `push` : xcom_push(key=<key>, value=<value>)
   - `pull` : xcom_pull(key=<key>) 혹은 xcom_pull(task_ids=<task_id>) 두가지 방식

   ```python
   def xcom_push_test(**context):
       xcom_value = "xcom_push_value"
       context['task_instance'].xcom_push(key='xcom_push_value', value=xcom_value)
   
       return "xcom_return_value"
   
   def xcom_pull_test(**context):
       xcom_return = context["task_instance"].xcom_pull(task_ids='return_xcom')
       xcom_push_value = context['ti'].xcom_pull(key='xcom_push_value')
       xcom_push_return_value = context['ti'].xcom_pull(task_ids='xcom_push_task')
   
       print("xcom_return : {}".format(xcom_return))
       print("xcom_push_value : {}".format(xcom_push_value))
       print("xcom_push_return_value : {}".format(xcom_push_return_value))
       
   xcom_push_task = PythonOperator(
       task_id = 'xcom_push_task',
       python_callable = xcom_push_test,
       dag = dag
   )
   
   xcom_pull_task = PythonOperator(
       task_id = 'xcom_pull_task',
       python_callable = xcom_pull_test,
       dag = dag
   )
   ```

3. **Jinja templates을 이용**

   앞과 동일하게 `task_instance(ti)`를 이용하는 것과 동일하게 사용이 가능하며 push, pull 모두 사용이 가능

   ```python
   bash_xcom_taskids = BashOperator(
       task_id='bash_xcom_taskids',
       bash_command='echo "{{ task_instance.xcom_pull(task_ids="xcom_push_task") }}"',
       dag=dag
   )
   
   bash_xcom_key = BashOperator(
       task_id='bash_xcom_key',
       bash_command='echo "{{ ti.xcom_pull(key="xcom_push_value") }}"',
       dag=dag
   )
   
   bash_xcom_push = BashOperator(
       task_id='bash_xcom_push',
       bash_command='echo "{{ ti.xcom_push(key="bash_xcom_push", value="bash_xcom_push_value") }}"',
       dag=dag
   )
   
   bash_xcom_pull = BashOperator(
       task_id='bash_xcom_pull',
       bash_command='echo "{{ ti.xcom_pull(key="bash_xcom_push") }}"',
       dag=dag
   )
   ```

### Unit testing

Pytest : https://docs.pytest.org/en/latest

- DAG Validation Tests
  - 오타 체크
  - DAG가 acyle로 구성되어 있는지 체크
  - arguments가 올바르게 설정되어 있는지 체크
- DAG/Pipeline Definetion Tests
- Unit Tests
- Integration Tests
- End to End Pipeline Tests