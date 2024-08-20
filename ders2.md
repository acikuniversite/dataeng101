# Ders 2: Veri Kaynakları ve ETL Süreçleri

Bu derste, veri mühendisliği sürecinde karşılaşılan farklı veri kaynakları ve bu verilerin ETL (Extract, Transform, Load) süreçleri ile nasıl işleneceğini öğreneceksiniz.

## Ders 2a: Veri Kaynakları ve Veri Toplama

### Farklı Veri Kaynakları
Veri mühendisliği projelerinde çeşitli veri kaynaklarından veri toplanabilir. Bu kaynaklar şunları içerir:
- **RDBMS (İlişkisel Veritabanı Yönetim Sistemleri)**: SQL tabanlı veri tabanları (MySQL, PostgreSQL, Oracle) gibi yapılandırılmış verilerin saklandığı veri kaynakları.
- **NoSQL Veritabanları**: MongoDB, Cassandra gibi esnek veri modellerine sahip veritabanları.
- **API'ler**: Üçüncü parti hizmetlerden veri çekmek için kullanılan arayüzler.
- **Dosyalar**: CSV, JSON, XML gibi dosya formatlarında saklanan veriler.

### Veri Toplama Yöntemleri ve Araçları
Veri toplama sürecinde kullanılan bazı yaygın yöntemler ve araçlar:
- **Web Scraping**: Web sitelerinden veri çekmek için kullanılan bir tekniktir. Python'da BeautifulSoup ve Scrapy gibi kütüphaneler yaygın olarak kullanılır.
- **API Entegrasyonu**: RESTful API'ler aracılığıyla veri çekme işlemi yapılabilir. Python'da `requests` kütüphanesi bu işlem için sıkça kullanılır.

### Web Scraping ile Veri Toplama
Web scraping, belirli bir web sitesinden veri almak için kullanılan bir tekniktir. Aşağıda basit bir örnek verilmiştir:
```python
import requests
from bs4 import BeautifulSoup

url = 'https://example.com'
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# Örnek: Tüm başlıkları toplama
titles = soup.find_all('h1')
for title in titles:
    print(title.get_text())
```

### Selenium ile Dinamik Web Sitelerinden Veri Toplama
Selenium, dinamik web sitelerinden veri toplamak için kullanılan bir araçtır. Aşağıda basit bir örnek verilmiştir:
```python
from selenium import webdriver

url = 'https://example.com'
driver = webdriver.Chrome()

driver.get(url)
elements = driver.find_elements_by_tag_name('h1')

for element in elements:
    print(element.text)

driver.quit()
```

### API Entegrasyonu ile Veri Toplama
API'ler üzerinden veri çekme işlemi genellikle JSON formatında yapılır. Aşağıda basit bir API'den veri çekme ve JSON ile çalışma örneği verilmiştir:

```python
import requests
import json

url = 'https://api.example.com/data'
response = requests.get(url)
data = response.json()

# JSON verisini işleme
for item in data:
    print(item['field'])
```

---

## Ders 2b: ETL Süreçleri ve Veri Temizleme

### ETL Nedir?
ETL, veri mühendisliğinde sıkça kullanılan bir kısaltmadır ve şu anlamlara gelir:
- **Extract (Çıkarma)**: Veri kaynaklarından verinin çıkarılması.
- **Transform (Dönüştürme)**: Verinin temizlenmesi, düzenlenmesi ve analiz için hazırlanması.
- **Load (Yükleme)**: Verinin hedef veritabanına veya depolama alanına yüklenmesi.

### Verinin Temizlenmesi ve Dönüştürülmesi
Veri temizliği ve dönüşümü, ETL sürecinin kritik bir aşamasıdır. Bu aşamada, verideki hatalar düzeltilir, eksik veriler tamamlanır ve veriler analiz için uygun hale getirilir. Python'da Pandas kütüphanesi bu işlemler için sıklıkla kullanılır.

```python
import pandas as pd

# Data Extraction (Veri Çıkarma)
df = pd.read_csv('data.csv')

# Data Cleaning (Veri Temizleme)
df.fillna(0, inplace=True)

# Data Transformation (Veri Dönüşümü)
df['new_column'] = df['old_column'] * 2

# Data Loading (Veri Yükleme)
df.to_csv('cleaned_data.csv', index=False)
```

### Veri Temizleme İşlemleri
Veri temizleme sürecinde sıkça kullanılan bazı işlemler:

- **Eksik Verilerin Doldurulması**: `fillna()` metodu ile eksik veriler doldurulabilir.
- **Veri Türü Dönüşümleri**: `astype()` metodu ile veri türleri dönüştürülebilir.
- **Aykırı Değerlerin Temizlenmesi**: `clip()` veya filtreleme yöntemleri ile aykırı değerler temizlenebilir.
- **Veri Birleştirme**: `merge()` veya `concat()` metotları ile veri birleştirme işlemi yapılabilir.
- **Veri Filtreleme**: Belirli koşullara uyan veriler filtrelenerek temizlenebilir.
- **Veri Normalizasyonu**: Verilerin belirli bir aralığa veya dağılıma getirilmesi.
- **Veri Dönüşümleri**: Verilerin logaritmik, üstel veya tersi alınarak dönüştürülmesi.
- **Veri Agregasyonu**: Verilerin gruplanarak toplanması veya özetlenmesi.
- **Veri Formatlama**: Verilerin belirli bir formata getirilmesi.
- **Veri Standardizasyonu**: Verilerin belirli bir ortalama ve standart sapmaya göre standartlaştırılması.
- **Veri Tokenizasyonu**: Metin verilerinin kelimelere veya sembollere ayrılması.
- **Veri Encoding**: Kategorik verilerin sayısal değerlere dönüştürülmesi.

### Apache Airflow ile ETL İşlemleri
Apache Airflow, ETL işlemlerini yönetmek ve zamanlamak için kullanılan güçlü bir araçtır. Airflow ile veri işleme süreçlerinizi otomatikleştirebilir ve izleyebilirsiniz.

#### Airflow Kurulumu
Airflow Scheduler ve Web Server şeklinde iki bileşenden oluşur. Aşağıdaki adımları izleyerek Airflow'u kurabilirsiniz:
1. Airflow'u pip ile yükleyin:
    ```bash
    pip install apache-airflow
    ```
   
2. Airflow veritabanını başlatın:
    ```bash
    airflow db init
    ```
   
3. Airflow Scheduler'ı başlatın:
    ```bash
    airflow scheduler
    ```
   
4. Airflow Web Sunucusu'nu başlatın:
    ```bash
    airflow webserver --port 8080
    ```
5. Kullanıcı adı ve şifre oluşturma.
    ```bash
    airflow users  create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin    
    ```
6. Dag klasörünü oluşturun ve config dosyasını güncelleyin.
    ```bash
    mkdir ~/airflow/dags
    ```
    ```bash
    airflow.cfg dosyasını güncelleyin.
    ```

7. Tarayıcınızda `http://localhost:8080` adresine giderek Airflow arayüzüne erişebilirsiniz.(kullanıcı adı ve şifre: `admin`)

#### Airflow ETL Pipeline Örneği
Aşağıda basit bir Airflow ETL pipeline örneği verilmiştir:

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.operators.dummy_operator import DummyOperator
from datetime import datetime

def etl_process():
    extract_data()
    transform_data()
    load_data()
    
def extract_data():
    print('Veri çıkarma işlemi...')
    
def transform_data():
    print('Veri dönüşüm işlemi...')
    
def load_data():
    print('Veri yükleme işlemi...')
    

dag = DAG('etl_pipeline', description='Veri ETL İşlemleri',
          schedule_interval='0 0 * * *',
          start_date=datetime(2022, 1, 1), catchup=False)

with dag:
    start = DummyOperator(task_id='start')
    end = DummyOperator(task_id='end')
    etl_task = PythonOperator(task_id='etl_process', python_callable=etl_process)

    start >> etl_task >> end
```

### Veri Kalitesi ve Güvenliği

Veri mühendisliğinde veri kalitesi ve güvenliği büyük önem taşır. Veri kalitesi, verinin doğruluğu, güvenilirliği ve tutarlılığı ile ilgilidir. Veri güvenliği ise verinin yetkisiz erişime karşı korunması ve gizliliğinin sağlanmasıdır. Veri kalitesi ve güvenliği sağlanmadığında, veri analizinde hatalar ve güvenlik riskleri ortaya çıkabilir.

