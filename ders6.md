# [Ders 6: Uçtan Uca Proje Uygulamaları](ders6.md)
## **Ders 6a: Uçtan Uca Proje 1 - Apache Airflow ile ETL Pipeline Geliştirme**
  - Proje Konusu: New York City Taksi Verisi Analizi
  - Proje Açıklaması ve Gereksinimler
    - Veri Seti: New York City Taksi Verileri (NYC Taxi Data), NYC Open Data platformundan indirilebilir.
    - Amaç: NYC taksi verisini işleyerek yolculuk sürelerini, mesafelerini ve ücretleri analiz edeceğiz. Veriyi ham halinden temizlenmiş ve analiz edilebilir hale getirmek için bir ETL pipeline geliştirilecek.
    - Gereksinimler: 
      - Donanımsal: Apache Airflow, AWS S3 veya SQL veritabanı
      - Yazılımsal: Python, Pandas, Jupyter Notebook
  - Adımlar:
    1. Veri Çekme: Airflow DAG (Directed Acyclic Graph) kullanarak NYC Open Data API'sinden veri çekilecek.
    2. Veri Temizleme: Python ve Pandas kullanarak verideki eksik ve hatalı veriler temizlenecek.
    3. Veri Dönüştürme: Veriler, yolculuk uzunluğu, ortalama ücret gibi metriklere dönüştürülecek.
    4. Veri Yükleme: Dönüştürülmüş veri, AWS S3 veya bir SQL veritabanına yüklenecek.
    5. Sonuçların Analizi: Analizler, Jupyter Notebook kullanılarak yapılacak ve görselleştirilecek.
    6. Raporlama: Analiz sonuçları Airflow ile otomatik olarak raporlanacak ve email ile gönderilecek.
  - Pratik Uygulama: Airflow ile ETL pipeline'ın adım adım kurulması ve yönetilmesi.

## Kod:

```python
from datetime import datetime, timedelta

from airflow import DAG

from airflow.operators.dummy_operator import DummyOperator

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2022, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'nyc_taxi_etl',
    default_args=default_args,
    description='NYC Taxi Data ETL Pipeline',
    schedule_interval=timedelta(days=1),
)

start = DummyOperator(task_id='start', dag=dag)
end = DummyOperator(task_id='end', dag=dag)

def extract_data():
  //https://data.cityofnewyork.us/resource/4b4i-vvec.json?$where=tpep_pickup_datetime%20between%20%272023-10-12T00:00:00%27%20and%20%272023-10-13T23:59:59%27
  url = 'https://data.cityofnewyork.us/resource/4b4i-vvec.json'
  todayLastYear = datetime.now() - timedelta(days=365)
  
  params = {
      '$where': f"tpep_pickup_datetime between '{todayLastYear.strftime('%Y-%m-%d')}T00:00:00' and '{todayLastYear.strftime('%Y-%m-%d')}T23:59:59'",
  }
   
  response = requests.get(url, params=params)
  data = response.json()
  return data
  
def transform_data(data):
    df = pd.DataFrame(data)
    # Eksik veya hatalı verileri temizleme
    df.dropna(inplace=True)
    df['trip_duration'] = pd.to_datetime(df['dropoff_datetime']) - pd.to_datetime(df['pickup_datetime'])
    # Temizlenmiş veriyi döndürme
    return df.to_dict(orient='records')
    
def load_data(data):
    engine = create_engine('sqlite:///nyc_taxi.db')
    df = pd.DataFrame(data)
    
    df.to_sql('nyc_taxi', con=engine, if_exists='replace', index=False)
    
    return len(df)
    
extract_task = PythonOperator(
    task_id='extract_data',
    python_callable=extract_data,
    dag=dag,
)

transform_task = PythonOperator(
    task_id='transform_data',
    python_callable=transform_data,
    provide_context=True,
    dag=dag,
)

load_task = PythonOperator(
    task_id='load_data',
    python_callable=load_data,
    provide_context=True,
    dag=dag,
)

start >> extract_task >> transform_task >> load_task >> end
```

## **Ders 6b: Uçtan Uca Proje 2 - Apache Spark ve Google BigQuery ile Büyük Veri Analizi
  - Proje Konusu: Twitter Veri Analizi
  - Proje Açıklaması ve Gereksinimler
    - Veri Seti: Twitter verisi (belirli bir hashtag veya anahtar kelime ile ilgili tweet'ler), Twitter API'sinden çekilebilir. Twitter Developer hesabı gereklidir.
    - Amaç: Belirli bir hashtag ile ilgili tweet'ler toplanarak, duygu analizi (sentiment analysis) ve kelime frekansı analizi yapılacak. Veriyi işlemek ve analiz etmek için Apache Spark kullanılacak, sonuçlar Google BigQuery üzerinde saklanacak ve sorgulanacak.
    - Gereksinimler: 
      - Donanımsal: Apache Spark Cluster, Google Cloud Platform hesabı
      - Yazılımsal: Apache Spark, Google Cloud SDK, Twitter API erişimi
  - Adımlar:
    1. Veri Çekme: Twitter API kullanılarak belirli bir zaman aralığındaki tweet'ler çekilecek ve Google Cloud Storage'a yüklenecek.
    2. Veri İşleme: Apache Spark kullanarak, tweet'lerdeki metinler analiz edilecek ve duygu analizi yapılacak.
    3. Veri Dönüştürme: Tweet verisi, duygu skorları ve kelime frekansı gibi metriklere dönüştürülecek.
    4. Veri Yükleme: Sonuçlar, Google BigQuery'ye yüklenerek sorgulama için optimize edilecek.
    5. Sonuçların Analizi: BigQuery kullanılarak SQL sorguları ile veriler analiz edilecek ve sonuçlar görselleştirilecek.
    6. Raporlama: Analiz sonuçları Tableau veya Google Data Studio ile görselleştirilerek raporlanacak.ü
  - Pratik Uygulama: Spark ile büyük veri analizi yapılması ve BigQuery ile veri sorgulanması.
### Adım Adım
#### 0. Gereksinimler
```bash
pip install pyspark
pip install google-cloud-bigquery
pip install requests
pip install tweepy
```
#### 1. Veri Çekme
Twitter API kullanarak tweet'leri çekme:
```python
import tweepy
import json
from pyspark.sql import SparkSession

# Twitter API kimlik bilgilerini girin
api_key = 'YOUR_API_KEY'
api_secret_key = 'YOUR_API_SECRET_KEY'
access_token = 'YOUR_ACCESS_TOKEN'
access_token_secret = 'YOUR_ACCESS_TOKEN_SECRET'

# Twitter API'ye bağlanma
auth = tweepy.OAuthHandler(api_key, api_secret_key)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)

# Belirli bir hashtag ile ilgili tweet'leri çekme
def extract_tweets(hashtag, count=100):
    tweets = api.search(q=hashtag, count=count, lang='en', tweet_mode='extended')
    tweet_data = []
    for tweet in tweets:
        tweet_data.append({
            'id': tweet.id_str,
            'created_at': tweet.created_at,
            'text': tweet.full_text,
            'user': tweet.user.screen_name,
            'location': tweet.user.location,
            'retweet_count': tweet.retweet_count,
            'favorite_count': tweet.favorite_count
        })
    return tweet_data

# Örnek: #DataScience hashtag'i ile ilgili 100 tweet'i çekme
tweets = extract_tweets("#DataScience", 100)
```
#### Veriyi Spark ile İşleme (Transform)
Apache Spark kullanarak tweet'leri analiz edeceğiz. Bu adımda, tweet'lerin duygu analizini yapacağız ve kelime frekanslarını hesaplayacağız.
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, udf
from pyspark.sql.types import StringType
from textblob import TextBlob

# Spark oturumu başlatma
spark = SparkSession.builder.appName("TwitterSentimentAnalysis").getOrCreate()

# Tweet verisini Spark DataFrame'e dönüştürme
df = spark.createDataFrame(tweets)

# Duygu analizi fonksiyonu
def analyze_sentiment(text):
    analysis = TextBlob(text)
    return 'positive' if analysis.sentiment.polarity > 0 else 'negative' if analysis.sentiment.polarity < 0 else 'neutral'

# UDF kaydetme
sentiment_udf = udf(analyze_sentiment, StringType())

# Duygu analizini uygulama
df = df.withColumn("sentiment", sentiment_udf(col("text")))

# Sonuçları gösterme
df.show()
```
#### Sonuçları Google BigQuery'ye Yükleme (Load)
Google BigQuery'ye sonuçları yükleme:
```python
from google.cloud import bigquery

# Google BigQuery istemcisini başlatma
client = bigquery.Client()

# BigQuery'ye yüklemek için geçici bir JSON dosyası kaydetme
df.toPandas().to_json('tweets.json', orient='records', lines=True)

# BigQuery veri seti ve tablo adlarını ayarlama
dataset_id = 'your_dataset_id'
table_id = 'twitter_sentiment_analysis'

# BigQuery yükleme işi oluşturma
job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,
    autodetect=True
)

with open('tweets.json', "rb") as source_file:
    job = client.load_table_from_file(source_file, table_id, job_config=job_config)

# Yükleme işlemi tamamlanana kadar bekleyin
job.result()

# Yüklenen veriyi kontrol etme
table = client.get_table(table_id)
print(f"Loaded {table.num_rows} rows to {table_id}")

# Spark oturumunu kapatma
spark.stop()
```
#### Sonuçların Analizi ve Görselleştirilmesi
Google BigQuery üzerinde SQL sorguları ile verileri analiz edebilir ve sonuçları görselleştirebilirsiniz. Ayrıca, Tableau veya Google Data Studio gibi araçlarla raporlar oluşturabilirsiniz.
```sql
SELECT sentiment, COUNT(*) AS tweet_count
FROM `your_project_id.your_dataset_id.twitter_sentiment_analysis`
GROUP BY sentiment
ORDER BY tweet_count DESC;
```

## **Ders 6c: Uçtan Uca Proje 3 - Hadoop Ekosistemi ile Veri İşleme ve Analiz**
  - Proje Konusu: NASA HTTP Web Server Log Data Analizi
  - Proje Açıklaması ve Gereksinimler
    - Veri Seti: NASA HTTP Web Server Log Data verisi, NASA'nın web sunucusu loglarını içeren bir veri setidir. Bu veri, web sunucu trafiğini analiz etmek için kullanılabilir.
    - Amaç: Bu projede, Hadoop ekosistemini kullanarak büyük ölçekli web sunucu log verisini işleyip analiz edeceğiz. Verinin işlenmesi ve saklanması HDFS üzerinde yapılacak, MapReduce kullanılarak analiz gerçekleştirilecek.
    - Gereksinimler: 
      - Donanımsal: Hadoop Cluster (en az 3 düğüm), HDFS depolama alanı
      - Yazılımsal: Hadoop, Hive, Pig, Python veya R
  - Adımlar:
    1. Veri Yükleme: Web sunucu log verileri HDFS'e yüklenecek.
    2. Veri İşleme: Hadoop MapReduce kullanılarak, log verilerinde en çok erişilen sayfalar, belirli IP'lerden gelen istekler ve hata kodları gibi metrikler hesaplanacak.
    3. Veri Sorgulama: Hive kullanılarak veriler SQL benzeri sorgularla analiz edilecek.
    4. Sonuçların Görselleştirilmesi: Elde edilen sonuçlar, Python veya R kullanılarak görselleştirilecek.
    5. Sonuçların Sunumu: Hadoop ekosisteminde elde edilen sonuçlar raporlanacak ve sunulacak.
  - Pratik Uygulama: Hadoop ekosistemi ile büyük veri analizi yapılması ve sonuçların raporlanması.
### Adım Adım
#### 0. Gereksinimler
**Uyarı** : Proje veri setleri ve gereksinimlerine erişmek için gerekli izinlere sahip olmanız gerekebilir. Bazı projeler, üçüncü taraf API'lerine erişim gerektirebilir. Proje veri setlerini kullanmadan önce ilgili veri kaynaklarından izin almanız önemlidir.
**Uyarı 2**: Örnekleri gerçekleştirmek için belirli bir cost olabilir. Örneğin, bulut platformlarındaki kaynak kullanımı ücretlendirilebilir. Bu nedenle, projeleri gerçekleştirmeden önce maliyetleri dikkate almanız önemlidir.
