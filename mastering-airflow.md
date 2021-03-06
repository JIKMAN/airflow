# Airflow-study

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/edefa2d6-55b8-489a-9c11-831aa5476cde/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T140230Z&X-Amz-Expires=86400&X-Amz-Signature=a13543f91a455cfc404a696748ecaede52211828421b7f8bdc0285181ab09701&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject" alt="untitled" style="zoom:67%;" />

## ๐กmastering DAGs

### execution_date ์ ์ดํด

> ***start_date** โ date at which DAG will start being scheduled

**schedule_interval** โ the interval of time from the minimum start_date at which we want our DAG to be triggered.*

> 

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9a8d7efa-5c6f-4393-a5f4-c19eb6b4af6a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132913Z&X-Amz-Expires=86400&X-Amz-Signature=8a6844ea7ed6c7315550c0f88bd46d7df5124f8c1e7fcdcd88d034a9f0095523&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

> **start_date๋ณด๋ค ๋ฆ๊ฒ DAG๊ฐ ์คํ๋๋ ์ด์ **
>
> airflow์ ์ปจ์์ด๋ผ๊ณ  ์๊ฐํ๋ฉด ๋  ๊ฒ ๊ฐ๋ค. ์๋ฅผ ๋ค์ด, ๋งค์ผ ํ๋ฃจ์น์ ๋ฐ์ดํฐ๋ฅผ ์์์ ์ฒ๋ฆฌํ๋ ์๋น์ค๊ฐ ์๋ค๊ณ  ๊ฐ์ ํ์ ๋  start_date(5์ 5์ผ 00์)๋ ์๋น์ค๊ฐ ์์๋ ๋ ์ด๋ค. ์ดํ 24์๊ฐ ๋์ ์์ธ ๋ฐ์ดํฐ๋ฅผ ๋ฐํ์ผ๋ก ์ค์ ๋ก ๋ฐ์ดํฐ ์ฒ๋ฆฌ ์์์ด ์ฒ์ ๋ฐ์ํ ์๊ฐ(์ฒซ ๋ฒ์งธ `DagRun` )์ 5์ 6์ผ 00์๊ฐ ๋๋ค.

```python
DAG = {
...
'start_date': datetime(2022, 5, 5),
schedule_interval="@daily",
...
}
```

- DAG๋ `2022-05-06`์ ์ต์ด ์คํ๋๋ฉฐ, `execution_date`๋ `2022-05-05`

  ์ฒซ ๋ฒ์งธ `DagRun`์ trigger๋ 5์ 5์ผ ์ดํ ์์  = 5์ 6์ผ 00์๊ฐ ๋๋ค. ์ฒซ ๋ฒ์งธ `DagRun`์ด ์คํ ๊ฐ๋ฅํ time range๋ 5์ 6์ผ 00 ~ 5์ 7์ผ 00์ (24hour, `@daily`)

- `2022-05-07` ์ ์คํ๋ DagRun์ `execution_date`๋ `2022-05-06`,

- `2022-05-08` ์ ์คํ๋ DagRun์ `execution_date`๋ `2022-05-07`,

  ์ฆ, execution_date๋ ์ค์  ์์์ runtime๊ณผ๋ ๋ฌด๊ดํ ํด๋น ์์์ ๋ํ๋ด๋ **id๊ฐ**์ด๋ผ๊ณ  ์๊ฐํ๋ฉด ๋๋ค.

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4de9aa96-9029-4dcb-9389-3a5764350ccf/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132932Z&X-Amz-Expires=86400&X-Amz-Signature=21b37ba50695cf0426d8d244c70ed3cb4c1bcafdfd2f7409019657ae38601c0c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

```python
some_bash_cmd = f"cd {scripts_folder}" + " && ./start.sh some_daily '{{ yesterday_ds }}'"
```

์ฐธ๊ณ ๋ก ์ค์ผ์ค๋ฌ๋ฅผ ํตํด ์คํ๋๋ DAG์ ๊ฒฝ์ฐ execution_date๊ฐ ์๋์ผ๋ก ํ ๋น๋๋ค.

CLI๋ฅผ ํตํด ๋ช๋ น์ด๋ก ์คํํ๋ ๊ฒฝ์ฐ execution_date๋ฅผ ๋ฐ๋์ ๋ช์ํด ์ฃผ์ด์ผ ํ๋ค.

```python
DAG = {
...
'start_date': datetime(2022, 5, 5),
schedule_interval="@hourly",
...
}
```

- execution_date๋ ์ค์  ์์์ runtime๊ณผ๋ ๋ฌด๊ด

| JOB No | DAG id  | execution_date    | start_date(runtime) | end_date          |
| ------ | ------- | ----------------- | ------------------- | ----------------- |
| 1      | zum_dag | 22-05-05T00:00:00 | 22-05-05T01:00:47   | 22-05-05T01:01:62 |
| 2      | zum_dag | 22-05-05T01:00:00 | 22-05-05T02:06:01   | 22-05-05T02:07:40 |
| 3      | zum_dag | 22-05-05T02:00:00 | 22-05-05T03:03:70   | 22-05-05T03:04:01 |

- ์ฐธ๊ณ ๋ก ์ฝ๋์์ DAG๋ฅผ ์ ์ธํ  ๋์ start_date๊ฐ์ DAG์ ์ ์ธ ์๊ฐ์ ๋ปํ์ง๋ง( ์ฌ๊ธฐ์  2022-05-05T00:00์ด ๋๋ค.), airflow scheduler UI์ start_date๋ ์ค์  job์ด ๋์ํ ์๊ฐ์ ๋งํ๋ค.

์ค์ผ์ค๋ง๋ daily ๋ฐฐ์น ์คํ ๋ง์ด ๋ชฉ์ ์ด๋ผ๋ฉด execution_date์ ์ฌ์ฉํ์ง ์๊ณ , ์ฝ๋ ๋ด์์ date_time.now()๋ฅผ ์ด์ฉ๊ฐ๋ฅ

ํ์ง๋ง ๋์ผํ ๋ฐฐ์น์ adhoc ํ ๋จ๋ฐ์ฑ ์คํ ์์๋ (i.e. backfill) ๋์ผํ dag ์ฝ๋๋ฅผ ํ์ฉํ๋ ค๋ฉด execution_date์ ์ฌ์ฉํ  ์๋ฐ์ ์๋ค.

๋ฐ๋ผ์ ์ ๋ ์ ์๋ฃ ๋๋ ์ด์  ์คํ ์๋ฃ๋ฅผ ์ด์ฉํด ETL์ ํ๊ณ  ์ถ์ ๊ฒฝ์ฐ์๋ yesterday_ds, prev_ds๋ฅผ ์ฌ์ฉํด์ผ ํ๋ค.

(ํ๋ฃจ ์ ๋ ์ ๋ก๊ทธ ๋ฐ์ดํฐ๋ฅผ ํตํด ETLํ๋ ์์์ด ํ์ํ๋ค๋ฉด, `yesterday_ds` ๋์  `ds`๋ฅผ ์ฌ์ฉ)

### (์ฐธ๊ณ ) timezone ์ค์ 

airflow๋ ๊ธฐ๋ณธ์ ์ผ๋ก **UTC timezone**์ ์ฌ์ฉํจ. ๋ฐ๋ผ์ korea timezone์ผ๋ก ๋ํ๋ด๊ธฐ ์ํด์๋ ์๊ฐ ์ค์  ๋ณ๊ฒฝ์ด ํ์ํจ.( ์ฐธ๊ณ  : Asia/Seoul์ UTC+09, ์ฐ๋ฆฌ๋๋ผ ์๊ฐ ๋๋น 9์๊ฐ ์  )

- **DAG์์ object๋ฅผ ํ์ฉ**

  airflow์์ TIMEZONE ๋ณํ์ ์ํด **pendulum** ํจํค์ง๋ฅผ ์ฌ์ฉํ  ์ ์๋ค.

  ```python
  import pendulum
  from datetime import datetime
  
  local_tz = pendulum.timezone("Asia/Seoul")
  
  default_args=dict(
      start_date = datetime(2020, 2, 28, 2, tzinfo=local_tz)
  ```

- **ํ๊ฒฝ ๋ณ์ ์ค์ **

  ```python
  # airflow.cfg
  ...
  #default_timezone = utc
  default_timezone = Asia/Seoul   <- Pendulum ๋ผ์ด๋ธ๋ฌ๋ฆฌ์์ ์ง์ํ๋ timezone ํํ
  ```

- **๋ค์ํ execution_date ํ๋ผ๋ฏธํฐ**

  ```python
  def _print_context(**context):
  		start=context["ececution_date"]
  		end=context["next_execution_date"]
  		print(f"Start: {start}, end: {end}")
  
  print_context=PythonOperator(
  		task_id="print_context", python_callable=_print_context, dag=dag
  )
  
  ## ์ถ๋ ฅ ์:
  ## Start: 2022-05-01T14:00:00+00:00, end 2022-05-01T15:00:00+00:00
  ```

  - airflow ์์ ์ ๊ณตํ๋ ๋ชจ๋  task context variable ๋ฐ jinja template

    https://airflow.apache.org/docs/apache-airflow/stable/templates-ref.html

  ```python
  ## Airflow 1.x   -> ์ฝํ์คํธ ๋ณ์ ์ฌ์ฉ์ ์ํด์๋ ์ต์์ ์ค์ ํด์ค์ผํจ.
  PythonOperator(
  		task_id="pass_context",
  		python_callable=_pass_context,
  		provide_context=True,
  		dag=dag
  		...
  ## Airflow 2.x   -> ์ฝํ์คํธ ๋ณ์์ ๋ํ ํธ์ถ ์ฌ๋ถ๋ฅผ ์๋์ผ๋ก ํ๋จํจ.
  PythonOperator(
  		task_id="pass_context",
  		python_callable=_pass_context,
  		dag=dag
  		...
  ```

### backfill and catchup

- Backfill

  ๊ณผ๊ฑฐ๋ถํฐ ํ์ฌ๊น์ง ์คํ๋์์ด์ผ ํ๋ DAG๋ฅผ ๋ค์ ์คํํ๋ ๊ณผ์ 

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2fa75d3f-db35-4e9e-8ab3-af028d262c21/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132951Z&X-Amz-Expires=86400&X-Amz-Signature=b09deab3e9122acd20dec644f657eae8ab86c3893cedad0ddec5bc4f4aa4ab18&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/11826649-1bb7-4bdd-9050-07223a9580a3/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T132959Z&X-Amz-Expires=86400&X-Amz-Signature=677424b5e216c57f3933f28bb344b9bb5ab7b1d0525ecfe4fcfc1611ec258658&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

CLI๋ฅผ ํตํด backfill์ ๋ค์ํ  ๊ฒฝ์ฐ

```bash
airflow backfill -s 2020-05-05 -e 2020-05-10 <dag_id>
```

backfill์ ๊ธฐ๋ณธ์ ์ผ๋ก ์คํ๋์ง ์์ Task๋ค๋ง ์คํํ๊ธฐ ๋๋ฌธ์ DAG์ ์์ ๋ค์ ์คํํ๊ณ  ์ถ๋ค๋ฉด `-reset_dagruns`๋ฅผ ๊ฐ์ด ์ธ์๋ก ์ค์ผ ํจ

```bash
airflow backfill -s 2020-01-05 -e 2020-01-10 --reset_dagruns <dag_id>
```

**์คํจํ ์์์ ํํด** backfill์ ์คํํด์ผ ํ๋ ๊ฒฝ์ฐ ๋ค์์ฒ๋ผ ์ฌ์ฉํ  ์ ์์. (catchup ํ๋ผ๋ฏธํฐ์ ๋ฌด๊ดํ๊ฒ ๋์)

```bash
airflow backfill -s 2020-01-05 -e 2020-01-10 --rerun_failed_tasks <dag_id>
```

- **Catch-up**

catchup ํ๋ผ๋ฏธํฐ๋ฅผ True๋ก ์ค์ ํ์ฌ backfill ์์์ด ์คํ๋๊ฒ ํ  ์ ์๋ค.

```python
with DAG(
		dag_id='my_dag',
		catchup=True,
) as dag:
```

### depends_on_past, wait_for_downstream

task์ ์ฑ๊ณต ์ฌ๋ถ์ ๋ฐ๋ผ ETL pipeline์ ์กฐ๊ฑด์ ์ฃผ๊ณ  ์ถ์ ๊ฒฝ์ฐ์ ์ฌ์ฉํ๋ ์ต์ ( ์ด์  task์ ์๊ด์์ด ์คํ ๊ฐ๋ฅํ ์์์ ๊ฒฝ์ฐ๋ ์ฌ์ฉํ์ง ์์. `default : False`)

- **dpends_on_past**

  ์ด์  ๋ ์ง์ task ์ธ์คํด์ค ์ค์์ ๋์ผํ task ์ธ์คํด์ค๊ฐ ์คํจํ ๊ฒฝ์ฐ ์คํ๋์ง ์๊ณ  ๋๊ธฐ.

  ```jsx
  default_args = {
      'start_date': datetime(2021, 5, 1),
      "retries": 0,
      "depends_on_past": True
  }
  ```

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e6f688d8-405e-4f1f-affb-3540541557c5/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T133012Z&X-Amz-Expires=86400&X-Amz-Signature=91e54fbe8a0b253160426de95d3aa130b8154352d6c00ee3e1691c3d8d900b0f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

- **wait_for_downstream**

  ์ด์  ๋ ์ง์ task ์ธ์คํด์ค ์ค ํ๋๋ผ๋ ์คํจํ ๊ฒฝ์ฐ์๋ ํด๋น DAG๋ ์คํ๋์ง ์๊ณ  ๋๊ธฐ.

  ```jsx
  default_args = {
      'start_date': datetime(2021, 5, 1),
      "retries": 0,
      "wait_for_downstream": True
  }
  ```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/efa2711a-0f16-45a3-905e-6fef40883d65/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T133026Z&X-Amz-Expires=86400&X-Amz-Signature=586eb5a6e1389ec8987a5d1edd35988b53d2b9de4a57360c6b4b28ae22b6d5ff&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

- ์ด์  DAGRUN์ ์์์ด ์๋ฃ๋๊ธฐ ์ ์ ๋ค์ DAGRUN์ด ์คํ๋์ง ์์. ( ๋ค์ DAGRUN์ด ์ด์  DAGRUN์ ๊ฒฐ๊ณผ๋ฅผ ์ฐธ์กฐํ๋ ๊ฒฝ์ฐ ์ ์ฉํจ.)

### Retry, Alerting

- dagrun_timeout : DAG๊ฐ ์คํ๋๋๋ฐ ๊ฑธ๋ฆฌ๋ ์ต๋ ์๊ฐ, timeout์ด ์ง๋๋ฉด ์๋ก์ดDagRun์ ์์ฑ

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
		on_failure_callback=dag_failure_alert, ## DAG ๋จ๊ณ์์์ ์คํ ์ต์
) as dag:
	
	task1 = DummyOperator(task_id="task1")
  task2 = DummyOperator(task_id="task2")
  task3 = DummyOperator(task_id="task3", on_success_callback=dag_success_alert)
																					 ## task ๋จ๊ณ์์์ ์คํ ์ต์
  task1 >> task2 >> task3
```

- ์ฐธ๊ณ ) ์ค๋์ ์ง slack notifications

  https://gist.github.com/ddelange/6e33f8f0df3a97d4a371d055aa2d58ac

- max_active_runs : ๋์์ ์คํ๋  ์ ์๋ DAGrun์ ์

- concurrency : ๋์์ ์คํ๋  ์ ์๋ task์ ์

- task_concurrency : ๋์์ ์คํ๋  ์ ์๋ ๋์ผํ task์ ์

  ![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9b20188b-2c10-4def-b06d-e74a129af3fc/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220510%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220510T133037Z&X-Amz-Expires=86400&X-Amz-Signature=b1291b6f22b5ef2e2cecc3d4c88f4d8530308977bd37b113ed91dfed4eb8cb81&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### Xcom

Airflow๋ ๊ธฐ๋ณธ์ ์ผ๋ก ํ๋์ task์ ๊ฒฐ๊ณผ๊ฐ ๋ค๋ฅธ task์ ์ํฅ์ ์ฃผ์ง ์๋๋ค. ๊ฐ๊ฐ์ด ๋๋ฆฝ์ ์ผ๋ก ์คํ๋๊ธฐ ๋๋ฌธ์ ์๋ก ํต์ ํ  ์๋จ์ด ์๋ค. ํ์ง๋ง workflow๋ฅผ ๋ง๋ค๋ค ๋ณด๋ฉด ์ด์  ์์์ ๊ฒฐ๊ณผ, ์์ ๋ฑ์ ๋ค์ ์์์ ์ ๋ฌํ  ๊ฒฝ์ฐ๊ฐ ์๊ธด๋ค. ์ด๋ Xcom์ ์ด์ฉํด ์ ๋ฌํ  ์ ์๋ค.

Xcom์ ์ด์ฉํด ๋ฐ์ดํฐ๋ฅผ ์ ๋ฌํ๋ ๊ฒฝ์ฐ DataFrame์ด๋ ๋ง์ ์์ ๋ฐ์ดํฐ๋ฅผ ์ ๋ฌํ๋ ๊ฒ์ ์ง์ํ์ง ์์ผ๋ฉฐ, ์๋์ ๋ฐ์ดํฐ๋ง ์ ๋ฌํ๋ ๊ฒ์ ๊ถ์ฅํ๋ค. (MAX XCOM size 48Kb)

Xcom์ ์ฌ์ฉํ๊ธฐ ์ํด์๋ ๊ฐ task์์ push, pull ํ๋ ๋ฐฉ์์ผ๋ก ๊ธฐ๋ณธ์ ์ผ๋ก ์ฌ์ฉ๋์ง๋ง, PythtonOperator์ ๊ฒฝ์ฐ return์ด ์๋์ ์ผ๋ก Xcom ๋ณ์๋ก ์ง์ ๋๊ฒ ๋๋ค.

- ๊ณต์๋ฌธ์

  https://airflow.apache.org/docs/apache-airflow/stable/concepts/xcoms.html?highlight=xcom

X**com ์ฌ์ฉ๋ฐฉ๋ฒ**

- PythonOperator return ๊ฐ์ ์ด์ฉํ Xcom ์ฌ์ฉ
- push-pull์ ์ด์ฉํ Xcom ์ฌ์ฉ
- Jinja templates์ ์ด์ฉํ Xcom ์ฌ์ฉ

1. **PythonOperator ์ด์ฉ**

   PythonOperator๋ฅผ ์ด์ฉํ๋ฉด return ๊ฐ์ด Xcom์ผ๋ก ์๋ push๋จ.

   ```python
   def return_xcom():
       return "xcom!"
       
   return_xcom = PythonOperator(
       task_id = 'return_xcom',
       python_callable = return_xcom,
       dag = dag
   )
   ```

2. **push-pull์ ์ด์ฉ**

   `context['task_instance']`๋ฅผ ์ด์ฉํ์ฌ xcom์ push, pull ํ์ฌ ๋ฐ์ดํฐ๋ฅผ ์ฃผ๊ณ ๋ฐ๋ ๋ฐฉ๋ฒ

   - `push` : xcom_push(key=<key>, value=<value>)
   - `pull` : xcom_pull(key=<key>) ํน์ xcom_pull(task_ids=<task_id>) ๋๊ฐ์ง ๋ฐฉ์

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

3. **Jinja templates์ ์ด์ฉ**

   ์๊ณผ ๋์ผํ๊ฒ `task_instance(ti)`๋ฅผ ์ด์ฉํ๋ ๊ฒ๊ณผ ๋์ผํ๊ฒ ์ฌ์ฉ์ด ๊ฐ๋ฅํ๋ฉฐ push, pull ๋ชจ๋ ์ฌ์ฉ์ด ๊ฐ๋ฅ

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
  - ์คํ ์ฒดํฌ
  - DAG๊ฐ acyle๋ก ๊ตฌ์ฑ๋์ด ์๋์ง ์ฒดํฌ
  - arguments๊ฐ ์ฌ๋ฐ๋ฅด๊ฒ ์ค์ ๋์ด ์๋์ง ์ฒดํฌ
- DAG/Pipeline Definetion Tests
- Unit Tests
- Integration Tests
- End to End Pipeline Tests