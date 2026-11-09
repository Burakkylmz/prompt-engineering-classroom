# Prompt Engineering Classroom — Öğrenci Dokümantasyonu

Bu doküman, `prompt-engineering-classroom` repository'sinde yer alan LAB01,
LAB02 ve LAB03 laboratuvarlarını bağımsız olarak çalıştırmak, kodunu okumak
ve deney sonuçlarını mühendislik açısından yorumlamak isteyen katılımcılar
için hazırlanmıştır. İçerik, repository'nin mevcut dosyalarına (notebook
kodları, `config/` dosyaları, `data/` veri setleri) dayanmaktadır; burada
anlatılmayan hiçbir özellik ya da sonuç uydurulmamıştır.

## İçindekiler

1. [Proje Hakkında](#1-proje-hakkında)
2. [Öğrenme Hedefleri](#2-öğrenme-hedefleri)
3. [Kullanılan Teknolojiler](#3-kullanılan-teknolojiler)
4. [Proje Klasör Yapısı](#4-proje-klasör-yapısı)
5. [Kurulum ve Çalıştırma](#5-kurulum-ve-çalıştırma)
6. [LAB01 — Prompt Anatomy](#6-lab01--prompt-anatomy)
7. [LAB02 — Instruction Hierarchy & Prompt Injection](#7-lab02--instruction-hierarchy--prompt-injection)
8. [LAB03 — Zero-Shot vs Few-Shot Prompting](#8-lab03--zero-shot-vs-few-shot-prompting)
9. [Değerlendirme Metrikleri](#9-değerlendirme-metrikleri)
10. [Deney Sonuçlarını Nasıl Yorumlamalıyız?](#10-deney-sonuçlarını-nasıl-yorumlamalıyız)
11. [Öğrenci Çalışma Rehberi](#11-öğrenci-çalışma-rehberi)
12. [Sık Yapılan Hatalar](#12-sık-yapılan-hatalar)
13. [Sonuç ve Kazanımlar](#13-sonuç-ve-kazanımlar)

---

## 1. Proje Hakkında

Bu repository klasik bir uygulama geliştirme projesi değildir. Burada bir
ürün inşa etmiyoruz; bunun yerine **kontrollü Prompt Engineering deneyleri**
tasarlayıp gerçek bir LLM üzerinde (Groq API aracılığıyla) çalıştırıyor ve
sonuçları ölçerek yorumluyoruz. Her lab, aynı görevi (bir müşteri destek
mesajını sınıflandırmak) farklı prompt tasarım kararlarıyla tekrar tekrar
çözdürür; amaç, "daha fazla talimat / daha fazla örnek / daha sıkı format
her zaman daha iyi sonuç mu verir?" gibi soruları varsaymak değil, gerçek
ölçümlerle cevaplamaktır.

Üç lab'in ortak mimarisi kasıtlı olarak minimaldir: `src/` klasörü, class
yapıları veya özel bir framework yoktur — sadece okunabilir, sınıfta canlı
olarak değiştirilebilen küçük Python fonksiyonları vardır. Tüm deney
mantığı doğrudan notebook içindedir
([`notebooks/lab01_prompt_anatomy.ipynb`](../notebooks/lab01_prompt_anatomy.ipynb),
[`notebooks/lab02_instruction_hierarchy.ipynb`](../notebooks/lab02_instruction_hierarchy.ipynb),
[`notebooks/lab03_zero_shot_few_shot.ipynb`](../notebooks/lab03_zero_shot_few_shot.ipynb)).

Bu eğitimi tamamlayan bir katılımcı şu mühendislik becerilerini kazanır:

- Bir LLM API çağrısının (chat completion) `system` / `user` / `assistant`
  mesaj yapısını kod seviyesinde kurma ve okuma.
- Prompt bileşenlerini (instruction, context, constraints, output contract,
  few-shot examples) birbirinden ayrıştırılmış, değiştirilebilir parçalar
  olarak tasarlama.
- Bir deneyde **tek bir değişkeni** değiştirip diğer her şeyi sabit tutarak
  (controlled experiment) gerçek bir karşılaştırma yapma.
- Classification Accuracy, Format Compliance, Token Usage ve Latency gibi
  metrikleri birbirinden bağımsız olarak ölçme ve yorumlama.
- Instruction Hierarchy ve Prompt Injection kavramlarını somut kod ve
  gerçek saldırı örnekleriyle değerlendirme.
- Few-Shot Example seçiminin (sayı, çeşitlilik, sıralama) model kararlarını
  nasıl etkileyebileceğini deneysel olarak gözlemleme.

## 2. Öğrenme Hedefleri

Bu eğitim tamamlandığında katılımcılar şunları yapabilir hale gelir:

- LLM API mesaj yapısını (`system`, `user`, `assistant` rolleri) ve bir
  chat completion isteğinin gerçek JSON gövdesini okuyup yorumlamak.
- Etkili bir **System Prompt** oluşturmak: identity, instruction, context,
  constraints ve **Output Contract** bileşenlerini ayrıştırmak.
- Prompt bileşenlerinin (instruction, context, constraints, output format,
  few-shot examples) her birinin tek başına etkisini ölçmek.
- **Instruction Hierarchy** kavramını — hangi içeriğin "talimat", hangisinin
  "veri" sayıldığını — kod seviyesinde ayırt etmek.
- **Prompt Injection** risklerini gerçek saldırı senaryolarıyla
  değerlendirmek ve bunlara karşı alınan önlemlerin sınırlarını anlamak.
- **Zero-Shot Prompting** ile **Few-Shot Prompting** yöntemlerini aynı
  görev üzerinde karşılaştırmak.
- **Example Selection** stratejilerinin (redundant/biased vs diverse,
  sıralama) etkisini incelemek.
- **Label Accuracy**, **Format Compliance**, **Token Usage** ve **Latency**
  metriklerini birbirinden bağımsız biçimde yorumlamak ve bu metriklerin
  ne zaman yanıltıcı olabileceğini fark etmek.

## 3. Kullanılan Teknolojiler

Aşağıdaki liste, `pyproject.toml`, `Dockerfile` ve `compose.yaml`
dosyalarından doğrulanmıştır.

| Teknoloji | Sürüm / Kaynak | Kullanım Amacı |
|---|---|---|
| Python | `>=3.10,<3.11` (`pyproject.toml`), Docker imajında `python:3.10-slim` | Çalışma zamanı |
| [uv](https://docs.astral.sh/uv/) | `ghcr.io/astral-sh/uv:latest` (Docker), yerelde kurulu `uv` | Bağımlılık yönetimi ve sanal ortam (`.venv`) oluşturma |
| `groq` | `>=0.11.0` | Groq API istemcisi (chat completion çağrıları) |
| `pandas` | `>=2.2.0` | Deney sonuçlarını `DataFrame` olarak işlemek, `groupby().agg()` ile metrik hesaplamak |
| `matplotlib` | `>=3.9.0` | Sonuçların çubuk grafiklerle görselleştirilmesi |
| `pyyaml` | `>=6.0.0` | `config/*.yaml` prompt konfigürasyon dosyalarını okumak |
| `tomli` | `>=2.0.0` | `config/settings.toml` dosyasını okumak |
| `python-dotenv` | `>=1.0.0` | `.env` dosyasından `GROQ_API_KEY`'i yüklemek |
| `jupyterlab` | `>=4.2.0` | Notebook arayüzü |
| `ipykernel` | `>=6.29.0` | Jupyter kernel desteği |
| Docker / Docker Compose | `Dockerfile`, `compose.yaml` | Opsiyonel container tabanlı çalışma ortamı |
| Jupyter Notebook (`.ipynb`) | — | Her lab'in tek dosyalık deney ortamı |

Model sağlayıcısı olarak **Groq API** kullanılır; kullanılan model adı ve
üretim parametreleri `config/settings.toml` içinde tanımlıdır (bkz.
Bölüm 5).

## 4. Proje Klasör Yapısı

Repository'nin gerçek yapısı (geliştirme sırasında oluşan `.venv/` ve
`.ipynb_checkpoints/` gibi otomatik/üretilen klasörler hariç tutulmuştur):

```text
prompt-engineering-classroom/
├── notebooks/
│   ├── lab01_prompt_anatomy.ipynb
│   ├── lab02_instruction_hierarchy.ipynb
│   └── lab03_zero_shot_few_shot.ipynb
├── config/
│   ├── settings.toml          # Model ve deney parametreleri (LAB01/LAB02 ortak)
│   ├── prompts.yaml           # LAB01 prompt bileşenleri
│   ├── lab02_prompts.yaml     # LAB02 prompt bileşenleri
│   └── lab03_prompts.yaml     # LAB03 prompt bileşenleri
├── data/
│   ├── messages.json          # LAB01 değerlendirme veri seti (15 mesaj)
│   ├── lab02_cases.json       # LAB02 test vakaları (14 vaka)
│   ├── lab03_evaluation.json  # LAB03 değerlendirme veri seti (20 mesaj)
│   └── lab03_examples.json    # LAB03 few-shot örnek havuzu (15 aday)
├── docs/
│   └── README.md              # Bu dokümantasyon
├── .env.example                # GROQ_API_KEY şablonu
├── Dockerfile                  # JupyterLab container imajı
├── compose.yaml                # Docker Compose servis tanımı
├── pyproject.toml              # Python bağımlılıkları (uv)
├── uv.lock                      # Kilitlenmiş bağımlılık sürümleri
└── README.md                   # Repository kök dokümantasyonu
```

Klasörlerin işlevsel ayrımı:

- **`notebooks/`**: Deneylerin gerçekten çalıştırıldığı yer. Her dosya
  kendi başına çalışabilen, bağımsız bir lab'dir.
- **`config/`**: Prompt metinleri ve model/deney parametreleri. Kod
  değiştirilmeden, sadece bu dosyalar düzenlenerek farklı prompt
  varyasyonları test edilebilir.
- **`data/`**: Deneylerde kullanılan sentetik (gerçek müşteri verisi
  içermeyen) Türkçe test vakaları ve few-shot örnek havuzları.
- **`docs/`**: Bu eğitim dokümantasyonunun bulunduğu klasör.

## 5. Kurulum ve Çalıştırma

### Gereksinimler

- Python 3.10 (`pyproject.toml`: `requires-python = ">=3.10,<3.11"`)
- [uv](https://docs.astral.sh/uv/) (yerel kurulum için)
- Docker ve Docker Compose (opsiyonel, container kurulumu için)
- Bir Groq API key ([console.groq.com](https://console.groq.com)) —
  yalnızca gerçek (canlı) deneyleri çalıştırmak için gereklidir

### Repository'yi Klonlama

```bash
git clone <repository-url>
cd prompt-engineering-classroom
```

### Yerel Kurulum (uv ile)

```bash
uv sync
uv run jupyter lab
```

`uv sync`, `pyproject.toml` / `uv.lock` dosyalarındaki tüm bağımlılıkları
kullanarak bir `.venv` oluşturur. `uv run jupyter lab` bu ortamı kullanarak
JupyterLab'i başlatır. Açılan arayüzde
`notebooks/lab01_prompt_anatomy.ipynb` dosyasını açarak başlayın.

### Docker ile Çalıştırma

```bash
docker compose up --build
```

JupyterLab, container içinde başlar ve `http://127.0.0.1:8888` adresinden
(yalnızca localhost'a bağlı) erişilebilir; erişim token'ı için container
loglarına bakın. `notebooks/`, `config/` ve `data/` klasörleri volume
olarak bağlanır, bu nedenle yerel düzenlemeler container içine anında
yansır. Aynı container hem LAB01 hem LAB02 hem de LAB03 için kullanılır;
ayrı bir Dockerfile veya servis gerekmez.

### Groq API Key Yapılandırması

```bash
cp .env.example .env
```

`.env` dosyasını açıp gerçek key'inizi girin:

```dotenv
GROQ_API_KEY=your_actual_key
```

`.env` dosyası git-ignored'dır ve notebook çıktısına hiçbir şekilde
yazdırılmaz. Docker için aynı değişken, kabuk (shell) ortamınızdan
`compose.yaml` aracılığıyla container'a aktarılır.

> **Önemli:** `GROQ_API_KEY` değerinizi veya `.env` dosyanızı **hiçbir
> zaman** GitHub'a (commit, push, gist, PR açıklaması vb.) göndermeyin.
> Key sızarsa Groq konsolundan hemen iptal edip yeni bir key oluşturun.

### Jupyter Notebook Erişimi ve Çalıştırma Sırası

Önerilen sıra: `LAB01` → `LAB02` → `LAB03`. Her notebook hücreleri **yukarıdan
aşağıya** çalıştırılmalıdır (`Run All` veya tek tek). Üç lab'de de aynı
güvenlik modeli uygulanır:

- Notebook ilk açıldığında **hiçbir ücretli API çağrısı otomatik olarak
  yapılmaz**.
- Gerçek (ücretli) çağrılar yalnızca ilgili hücrede `RUN_LIVE = True`
  bilinçli olarak ayarlandığında gerçekleşir. `RUN_LIVE = False`
  bırakıldığında notebook'un tüm yapısı, maliyetsiz biçimde incelenebilir.

## 6. LAB01 — Prompt Anatomy

**Notebook:** [`lab01_prompt_anatomy.ipynb`](../notebooks/lab01_prompt_anatomy.ipynb)
· **Config:** `config/prompts.yaml` · **Veri:** `data/messages.json`

### Amaç

LAB01, aynı sınıflandırma görevine (bir elektronik şirketinin müşteri
destek mesajını kategori + öncelik olarak etiketlemek) dört farklı prompt
yapısı uygulayarak **"daha fazla prompt bileşeni, her zaman daha yüksek
performans anlamına gelmez"** hipotezini test eder.

### Prompt Anatomy Bileşenleri

`config/prompts.yaml` içinde ayrıştırılmış ve `build_prompt()` fonksiyonu
tarafından sırayla birleştirilen bileşenler:

- **Identity / Instruction**: "Sen bir elektronik şirketinin müşteri
  destek mesajlarını sınıflandıran bir asistansın..."
- **Context**: Şirketin ne yaptığına ve müşterilerin hangi konularda
  yazdığına dair arka plan bilgisi.
- **Constraints**: Kategori ve öncelik değerlerinin kapalı küme (sadece 5
  kategori, sadece 3 öncelik) olduğunu belirten kurallar.
- **Examples (Few-Shot Examples)**: Değerlendirme veri setinde
  **bulunmayan** 3 örnek mesaj-etiket çifti (data leakage önlemi).
- **Output Contract**: Yanıtın tam olarak `ETIKET: <category>` /
  `ONCELIK: <priority>` iki satırlık formatında olmasını zorunlu kılan
  talimat.

```python
def build_prompt(experiment_key):
    experiment = prompts_config["experiments"][experiment_key]
    parts = []
    for component in experiment["components"]:
        if component == "few_shot_examples":
            parts.append(format_few_shot_examples(prompts_config["few_shot_examples"]))
        else:
            parts.append(prompts_config[component].strip())
    return "\n\n".join(parts)
```

Bu lab, mesajı tek bir `user` mesajı içinde gönderir (ayrı bir `system`
mesajı kullanmaz); `build_full_prompt()` bu birleştirilmiş metne
sınıflandırılacak müşteri mesajını ekler.

### Deneyler

| Deney | Bileşenler | Amaç |
|---|---|---|
| `E01_baseline` | Sadece instruction | Modelin hiçbir ek yönlendirme olmadan görevi "anlayıp" çözüp çözemediğini ölçmek |
| `E02_context_constraints` | + context + constraints | Bağlam ve kısıtların eklenmesinin etkisini ölçmek; henüz kesin çıktı formatı yok |
| `E03_output_contract` | + exact output format | Kesin bir **Output Contract**'ın Format Compliance'a etkisini ölçmek |
| `E04_few_shot` | + few-shot examples | **Few-Shot Prompting**'in önceki tüm bileşenlere ek katkısını ölçmek |

Her deney için sabit tutulan unsurlar: model, `temperature`,
`max_completion_tokens`, değerlendirme veri seti (`data/messages.json`,
`config/settings.toml`'daki `max_cases` ile sınırlanan ilk vakalar) ve
tekrar sayısı (`repetitions`).

Ölçülen metrikler: `label_accuracy`, `priority_accuracy`,
`format_compliance_rate`, ortalama prompt/completion token sayısı,
ortalama client-side **Latency**.

### Sınıflandırma Senaryosu

`data/messages.json` içinde 15 sentetik Türkçe müşteri destek mesajı
bulunur; her biri önceden şu beş kategoriden birine ve üç öncelik
seviyesinden birine etiketlenmiştir:

- Kategoriler: `billing`, `technical_issue`, `cancellation`, `complaint`,
  `general_inquiry`
- Öncelik seviyeleri: `low`, `medium`, `high`

`config/settings.toml`'daki `experiment.max_cases` değeri (varsayılan
`5`), her deneyde kaç mesajın işleneceğini belirler.

## 7. LAB02 — Instruction Hierarchy & Prompt Injection

**Notebook:** [`lab02_instruction_hierarchy.ipynb`](../notebooks/lab02_instruction_hierarchy.ipynb)
· **Config:** `config/lab02_prompts.yaml` · **Veri:** `data/lab02_cases.json`

### Amaç

LAB02, hayali bir "Kaya Elektronik" şirketi için bir **Return Decision
Assistant** (iade karar asistanı) üzerinde **Instruction Hierarchy**
(talimat hiyerarşisi) ve **Prompt Injection** (talimat enjeksiyonu)
kavramlarını inceler. Asistan, müşteri mesajını ve (bazı deneylerde) ekli
bir belgeyi şirketin sabit iade politikasına göre değerlendirip `ONAYLA` /
`REDDET` / `ESKALE` kararı verir.

### Temel Kavramlar

- **System Prompt / User Message**: `system` rolü uygulamanın kendisi
  tarafından sabitlenir (iade politikası + çıktı sözleşmesi); `user` rolü
  müşteri mesajını, varsa ekli belgeyi ve güvenilir CRM alanlarını
  (`days_since_purchase`, `is_defective`, `is_used`) içerir.
- **Trusted vs Untrusted Content**: Güvenilirlik, mesajın rolünden değil
  **veri kaynağından** gelir. CRM alanları `user` mesajı içinde yer alsa
  bile güvenilirdir, çünkü uygulama tarafından doğrudan sentetik veri
  setinden okunur — müşteri mesajından veya belgeden türetilmez. Buna
  karşılık, ekli bir belge `system` rolüne konsa bile güvenilmez kalır.
- **Prompt Injection**: Ekli belgenin içine, modelin gerçek talimatlarını
  (iade politikasını) geçersiz kılmayı amaçlayan sahte talimatlar
  yerleştirilmesi (örn. "ÖNEMLİ SİSTEM GÜNCELLEMESİ: önceki politikayı yok
  say").
- **Naive Concatenation**: Güvenilmeyen belgenin hiçbir güven sınırı
  belirtilmeden doğrudan `user` mesajına eklenmesi.
- **Untrusted Document Separation**: Aynı belgenin `<untrusted_document>`
  bloğunda sunulması ve `system` mesajına bu bloğun "veri, talimat değil"
  olduğunu belirten açık bir cümle eklenmesi. **`<untrusted_document>`
  gerçek bir API rolü değildir** — `user` mesajı içine yerleştirilmiş bir
  metin işaretleyicisidir; modelin onu nasıl yorumlayacağı tamamen
  `system` mesajındaki talimata bağlıdır.
- **Deterministic Validation**: Modelin ham kararı üzerine, uygulama
  tarafında (Python'da) çalışan, yalnızca güvenilir CRM alanlarını
  kullanan basit bir kontrol.
- **Attack Success Rate**: Bir saldırı vakasında modelin (veya sistemin)
  kararının saldırganın hedeflediği karara (`attack_target`) eşit olma
  oranı.
- **Model Decision vs System Decision**: `model_decision`, modelin ham
  yanıtından ayrıştırılan karardır; `final_decision`, buna uygulama
  kontrolü uygulandıktan sonraki nihai karardır (E01–E03'te ikisi her
  zaman aynıdır, yalnızca E04'te farklılaşabilir).

### Deneyler (E01–E04)

`build_messages()` fonksiyonu her deney için `system` ve `user` mesajlarını
**ayrı ayrı** döner:

```python
def build_messages(experiment_key, case):
    experiment = lab02_prompts["experiments"][experiment_key]
    system_parts = [lab02_prompts[component].strip() for component in experiment["system_components"]]
    system_message = "\n\n".join(system_parts)

    template = lab02_prompts["user_templates"][experiment["user_template"]]
    user_message = template.format(
        customer_message=case["customer_message"],
        document=case["document"] if experiment["uses_document"] else "",
        days_since_purchase=case["days_since_purchase"],
        is_defective=case["is_defective"],
        is_used=case["is_used"],
    )
    return [
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
```

| Deney | Ne değişiyor | `system` bileşenleri | Belge kullanımı |
|---|---|---|---|
| `E01_baseline` | Saldırı içermeyen normal talepler, ek belge yok | `policy`, `decision_contract` | Hayır |
| `E02_naive_concatenation` | Ek belge, hiçbir güven sınırı olmadan doğrudan birleştirilir | `policy`, `decision_contract` (E01 ile birebir aynı) | Evet |
| `E03_untrusted_separation` | Aynı belgeler, ama `<untrusted_document>` bloğu + açık güven talimatı ile | `policy`, `untrusted_content_instruction`, `decision_contract` | Evet |
| `E04_deterministic_validation` | E03 ile **tamamen aynı** prompt; API çağrısı tekrarlanmaz, E03'ün ham yanıtı yeniden kullanılır; üzerine deterministik kontrol uygulanır | E03 ile aynı | Evet (E03'ten devralınır) |

Notebook, E03 ve E04'ün gerçekten birebir aynı prompt'u ürettiğini bir
`assert` ile programatik olarak doğrular. `E04`'ün "ne değiştiği", sadece
`apply_deterministic_check()` fonksiyonunun çalıştırılıp çalıştırılmadığıdır:

```python
def apply_deterministic_check(is_defective, days_since_purchase, model_decision):
    if not is_defective and days_since_purchase > 30:
        return "REDDET"
    return model_decision
```

Bu kontrol **kasıtlı olarak çok dar kapsamlıdır**: yalnızca "ürün arızalı
değil ve satın almadan 30 günden fazla zaman geçmiş" ihlalini yakalar.
Başka hiçbir saldırı türünü (örneğin `ESKALE` hedefli saldırıları veya
30 günden az sürede yapılan sahte onay taleplerini) engellediğini iddia
etmez; bu, **Deterministic Validation**'ın kapsamını genel bir güvenlik
garantisi gibi sunmamak için kasıtlı bir tasarım kararıdır.

### Rol Yapısı ve `<untrusted_document>` Üzerine Önemli Not

`system`, `user` ve `assistant` rolleri Groq/OpenAI tarzı chat completion
API'lerinin standart mesaj rolleridir: `system` uygulamanın sabit
talimatını taşır, `user` kullanıcıdan (veya kullanıcı aracılığıyla gelen
içerikten) gelen girdiyi taşır, `assistant` modelin önceki yanıtlarını
temsil eder (LAB02'de kullanılmaz, LAB03'te few-shot örneklerde
kullanılır — bkz. Bölüm 8). `<untrusted_document>` bu üçünden **biri
değildir**; sadece `user` mesajının metni içine yerleştirilmiş bir
sınırlayıcıdır ve metinsel bir etiketten ibarettir — **tek başına bir
güvenlik garantisi sağlamaz**. Modelin bu etikete saygı göstermesi,
tamamen `system` mesajındaki açık talimata (`untrusted_content_instruction`)
bağlıdır; model bu talimatı yine de görmezden gelebilir.

## 8. LAB03 — Zero-Shot vs Few-Shot Prompting

**Notebook:** [`lab03_zero_shot_few_shot.ipynb`](../notebooks/lab03_zero_shot_few_shot.ipynb)
· **Config:** `config/lab03_prompts.yaml` · **Veri:**
`data/lab03_evaluation.json` (değerlendirme, 20 mesaj), `data/lab03_examples.json`
(few-shot örnek havuzu, 15 aday)

### Amaç

LAB03, **Zero-Shot Prompting** ile **Few-Shot Prompting** (**In-Context
Learning**) arasındaki farkı ve few-shot örnek seçiminin (sayı,
**Example Diversity**, temsil gücü, **Example Ordering**) model
performansı üzerindeki etkisini, bir e-ticaret Customer Support
Assistant'ının beş kategoriden birine mesaj sınıflandırması üzerinde
inceler. Bu lab'de **sadece kategori sınıflandırması** yapılır; LAB01'deki
öncelik sınıflandırması dahil edilmemiştir, böylece Example Selection
etkisi olabildiğince bağımsız ölçülür.

### Kavramlar

- **Zero-Shot Prompting**: Modele görevi yalnızca talimatlarla açıklamak;
  hiçbir örnek giriş-çıkış çifti göstermemek.
- **Few-Shot Prompting / In-Context Learning**: Gerçek soru sorulmadan
  önce, modele birkaç örnek giriş-çıkış çifti göstermek. Bu, model
  ağırlıklarını güncelleyen bir **fine-tuning işlemi değildir** — örnekler
  yalnızca o tek istek için bağlam (context) olarak sunulur.
- **Example Quality**: Örneğin doğru etiketlenmiş olması (bu lab'de tüm
  örnekler doğru etiketlidir; quality sabit tutulur).
- **Example Diversity / Example Representativeness**: Örneklerin birbirinden
  farklı olması ve tüm kategorileri dengeli temsil etmesi.
- **Example Redundancy**: Örneklerin belirli bir kategoride yoğunlaşması
  (düşük çeşitlilik).
- **Example Ordering / Order Sensitivity**: Aynı örnek kümesinin modele
  hangi sırada sunulduğunun karar üzerindeki etkisi.
- **Data Leakage**: Few-shot örneklerinin, değerlendirme veri setindeki
  mesajlarla çakışmaması (aksi halde model "ezberlemiş" gibi
  görünebilir). Notebook bunu programatik olarak doğrular (bkz. aşağıda).

### `build_messages()` Fonksiyonunun Rolü

```python
def build_messages(experiment_key, case):
    experiment = lab03_prompts["experiments"][experiment_key]
    system_parts = [
        lab03_prompts["identity"].strip(),
        lab03_prompts["instruction"].strip(),
        lab03_prompts["category_definitions"].strip(),
        lab03_prompts["classification_rules"].strip(),
        lab03_prompts["output_contract"].strip(),
    ]
    system_message = "\n\n".join(system_parts)

    template = lab03_prompts["user_message_template"]
    messages = [{"role": "system", "content": system_message}]

    for example_id in experiment["example_ids"]:
        example = examples_by_id[example_id]
        messages.append({"role": "user", "content": template.format(message=example["message"])})
        messages.append({"role": "assistant", "content": f"ETIKET: {example['label']}"})

    messages.append({"role": "user", "content": template.format(message=case["message"])})
    return messages
```

Bu fonksiyon tüm deneylerde **aynı** `system` mesajını üretir (identity +
instruction + category_definitions + classification_rules +
output_contract); deneyler arasındaki **tek fark**, hangi `example_ids`
listesinin kullanıldığı ve bu listenin sırasıdır.

### Few-Shot Examples: `user`/`assistant` Çiftleri Olarak Temsil

Her few-shot example, gerçek sınıflandırma sorusundan **önce** `messages`
dizisine eklenen bir `user` (örnek müşteri mesajı) ve bir `assistant`
(doğru `ETIKET: <label>` yanıtı) mesaj çifti olarak temsil edilir. Bu,
modele "daha önce böyle bir soru sorulmuş ve böyle doğru cevaplanmış"
görünümü verir — ama hiçbir model ağırlığı güncellenmez; bu temsil sadece
o tek API isteği için geçerlidir. Notebook, `E01_zero_shot` (`[system,
user]`) ile `E03_diverse_few_shot` (`[system, user, assistant, user,
assistant, ..., user]`) için üretilen gerçek `messages` dizilerini Step 6/7'de
yan yana yazdırarak bu farkı doğrudan gösterir.

### Deneyler

| Deney | Örnek Sayısı | Seçim Stratejisi | Sıralama |
|---|---:|---|---|
| `E01_zero_shot` | 0 | Örnek yok | — |
| `E02_redundant_few_shot` | 5 | Taraflı/tekrarlayan (4 `billing` + 1 `technical_issue`) | Sabit |
| `E03_diverse_few_shot` | 5 | Seçilmiş/dengeli (5 kategoriden birer örnek) | Sabit |
| `E04_reversed_order` | 5 | E03 ile **birebir aynı** 5 örnek | **Ters çevrilmiş** |

Notebook, bu dört deney tasarımının gereksinimlerini (E01'de örnek yok,
E02/E03'te tam 5 örnek, E04 = `reversed(E03)`) `assert` ifadeleriyle
programatik olarak doğrular.

### Evaluation Dataset vs Example Dataset

- **Evaluation Dataset** (`data/lab03_evaluation.json`): Deneylerde
  **sınıflandırılacak** 20 mesaj; her kategoriden tam 4 tane, zorluk
  seviyesi (`difficulty`) ve kısa bir not içerir.
- **Example Dataset** (`data/lab03_examples.json`): Few-shot örnek
  **havuzu**; 15 aday örnek.

Notebook, bu iki veri setinin **hiçbir ortak mesaj içermediğini**
(tam eşleşme bazında) programatik olarak doğrular — bu, **Data Leakage**'i
önlemek içindir. Bu kontrol tam-eşleşme (exact match) tabanlı olduğu için,
küçük kelime farklılıklarıyla yazılmış "near-duplicate" mesajları
yakalayamayabilir; bu nedenle veri setleri ayrıca elle de farklı
ifadelerle hazırlanmıştır.

### E02–E03 ve E03–E04 Karşılaştırmalarının Metodolojik Sınırları

- **E02 vs E03**: Bu iki deney arasında yalnızca diversity değil,
  örneklerin **içeriği ve temsil gücü de birlikte** değişir. Bu nedenle
  aradaki performans farkını tek başına "diversity" etkisine bağlamak
  doğru değildir — E02 ve E03 birlikte farklı bir example selection
  stratejisini temsil eder.
- **E03 vs E04**: Aynı 5 örnekle yapılan tek bir sıralama karşılaştırması,
  **Order Sensitivity** hakkında kesin bir sonuç vermez; genel accuracy
  değişmese bile belirli vakalarda model kararının değişip değişmediğine
  bakmak gerekir (notebook bunu `df_order_sensitivity` tablosuyla ayrıca
  gösterir).

## 9. Değerlendirme Metrikleri

Aşağıdaki metrikler, hangi lab'de kullanıldıklarıyla birlikte listelenmiştir.
Her metrik için üç soru cevaplanmıştır: **Neyi ölçer?**, **Nasıl
yorumlanır?**, **Hangi durumlarda yanıltıcı olabilir?**

| Metrik | Kullanıldığı Lab | Neyi ölçer? | Nasıl yorumlanır? | Ne zaman yanıltıcı olabilir? |
|---|---|---|---|---|
| **Label Accuracy** (`label_accuracy`) | LAB01, LAB03 | Ayrıştırılan kategorinin (`parsed_label`) beklenen kategoriye (`expected_label`) eşit olma oranı. | Yüksek değer, modelin doğru sınıflandırma yaptığını gösterir. Hatalı API çağrıları otomatik olarak `False` sayılır. | Model semantik olarak doğru cevap verse bile, parser bu cevabı **Output Contract** dışında bir biçimde yazıldığı için ayrıştıramazsa (ör. serbest Türkçe açıklama), accuracy yapay olarak düşük çıkar — bu, modelin görevi anlamadığı anlamına gelmez. |
| **Priority Accuracy** (`priority_accuracy`) | LAB01 | Ayrıştırılan önceliğin beklenen önceliğe eşit olma oranı. | `label_accuracy`'den **tamamen ayrı** ölçülür; biri yüksek diğeri düşük olabilir. | Label Accuracy ile karıştırılmamalıdır; aynı yanıtta ikisi birbirinden farklı sonuç verebilir. |
| **Format Compliance Rate** (`format_compliance_rate`) | LAB01, LAB02, LAB03 | Yanıtın, belirtilen **Output Contract**'a (tam istenen satır sayısı, ekstra metin olmadan) harfiyen uyma oranı. | Classification/Decision Accuracy'den bağımsızdır: doğru sınıflandırılmış ama formata uymayan bir yanıt mümkündür (ör. `"billing, high"`). | `E01_baseline` gibi prompt'ta hiç Output Contract istenmeyen deneylerde düşük Format Compliance beklenen bir durumdur; bunu bir "instruction-following başarısızlığı" gibi yorumlamak hatalıdır. |
| **Invalid Response Rate** (`invalid_response_rate`) | LAB02, LAB03 | API çağrısı başarılı oldu ama yanıt ayrıştırılamadı (`None`). | Bir **API hatasıyla aynı şey değildir**; model geçerli bir yanıt döndürmüş olabilir, sadece tanınan kategorilerden birini içermiyordur. | API Error Rate ile karıştırılırsa, "modelin çöktüğü" ile "modelin beklenmeyen bir format ürettiği" birbirine karışır. |
| **API Error Rate** (`api_error_rate`) | LAB02, LAB03 | API çağrısının kendisinin (ağ, rate limit, vb. nedenlerle) başarısız olma oranı. | `error` alanı `None` değilse bu satır hataya sayılır; dogruluk/format hesaplarında otomatik `False` kabul edilir. | Küçük veri setlerinde tek bir geçici ağ hatası oranı büyük ölçüde etkileyebilir; tekrar denenmeden kesin sonuç çıkarılmamalıdır. |
| **Prompt Tokens / Completion Tokens / Total Tokens** (`avg_prompt_tokens`, `avg_completion_tokens`, `avg_total_tokens`) | LAB01, LAB02, LAB03 | Groq API'den dönen gerçek **Token Usage** (asla tahmini değil). | Daha fazla bileşen (context, constraints, few-shot examples) eklemenin prompt token sayısını nasıl artırdığını somut olarak gösterir. | Token kullanımının artması, otomatik olarak accuracy'nin de arttığı anlamına gelmez; bu ikisi ayrı ayrı değerlendirilmelidir (bkz. Bölüm 10). |
| **Average Latency** (`avg_latency_ms` / `avg_api_latency_ms`) | LAB01, LAB02, LAB03 | `time.perf_counter()` ile client-side ölçülen, isteğin gönderilmesinden yanıtın tamamlanmasına kadar geçen süre. | Deneyler arası kaba bir karşılaştırma sağlar. | Tek bir çalıştırmadaki latency, ağ koşulları ve sunucu tarafı yüke bağlı olarak büyük ölçüde değişebilir; **tekrarlı ölçüm yapılmadan** kesin bir sonuç çıkarılmamalıdır. |
| **Average Validation Latency** (`avg_validation_latency_ms`) | LAB02 (yalnızca `E04_deterministic_validation`) | `apply_deterministic_check()` fonksiyonunun çalışma süresi (saf Python, API çağrısı değil). | Diğer deneylerde `NaN`'dır çünkü uygulama kontrolü yoktur. | API latency ile karıştırılmamalıdır; bu çok daha küçük (mikrosaniye/milisaniye altı) bir süredir. |
| **Model-Level Decision Accuracy / System-Level Decision Accuracy** (`model_level_decision_accuracy`, `system_level_decision_accuracy`) | LAB02 | Sırasıyla modelin ham kararının (`model_decision`) ve uygulama kontrolü sonrası nihai kararın (`final_decision`) `expected_decision`'a eşit olma oranı. | E01–E03'te bu ikisi her zaman aynıdır (uygulama kontrolü yok); yalnızca E04'te farklılaşabilir. | `E01_baseline`, E02/E03'ten **farklı bir vaka kümesi** kullanır; bu nedenle E01'in accuracy'si E02/E03'ünkiyle doğrudan karşılaştırılamaz. |
| **Model-Level / System-Level Attack Success Rate** (`model_level_attack_success_rate`, `system_level_attack_success_rate`) | LAB02 | Yalnızca saldırı vakaları (`is_attack == True`) üzerinden, kararın saldırganın hedefine (`attack_target`) eşit olma oranı. | Düşük değer daha iyi savunma anlamına gelir. `E01_baseline`'da hiç saldırı vakası olmadığı için bu metrik `NaN` olarak görünür — **`%0` DEĞİLDİR**. | `NaN`'ı `%0 saldırı başarısı` ile karıştırmak, "hiç saldırı yok" durumunu "tüm saldırılar engellendi" gibi yanlış yorumlamaya yol açar. |
| **Per-Category Accuracy** | LAB03 | Her deneyin, her bir kategori (`expected_label`) için ayrı ayrı accuracy'si. | Genel (ortalama) accuracy aynı görünse de, bazı kategorilerde sistematik hatalar olup olmadığını ortaya çıkarır. | Küçük `max_cases` değerlerinde bir kategoride çok az vaka olması, o kategorinin accuracy'sini (%0 veya %100 gibi) aşırı oynak hale getirebilir. |
| **Decision Change Rate** (`decision_change_rate`, `df_order_sensitivity`) | LAB03 (E03 vs E04) | Aynı vaka ve aynı örnek kümesiyle, sadece örnek sırası değiştiğinde model kararının değişme oranı. | Genel accuracy sabit kalsa bile belirli vakalarda kararın değişip değişmediğini gösterir — bu, Order Sensitivity'nin daha hassas bir ölçüsüdür. | `0` çıkması, modelin sıralamaya tamamen duyarsız olduğu anlamına gelmez; tek bir çalıştırma ve küçük veri setiyle genelleme yapılamaz. |

## 10. Deney Sonuçlarını Nasıl Yorumlamalıyız?

Bu bölümdeki örnekler, repository'de **kayıtlı** (notebook çıktılarında
saklı duran), gerçek bir `RUN_LIVE = True` çalıştırmasından alınmıştır.
Bunlar **tek bir çalıştırmanın** sonuçlarıdır; `temperature = 0` olsa bile
API çağrıları arasında tam determinizm garanti edilmez, dolayısıyla siz
aynı notebook'u çalıştırdığınızda farklı (hatta ters) sonuçlar
görebilirsiniz. Amaç "doğru sonucu ezberlemek" değil, **sonuçları nasıl
okuyacağınızı** öğrenmektir.

### Accuracy artışı olmadan token tüketiminin artması

Repository'de kayıtlı bir LAB03 çalıştırmasında dört deneyin de
`label_accuracy` değeri `1.0` (yani %100) olarak ölçülmüştür — buna
rağmen `avg_prompt_tokens` `E01_zero_shot` için `694.3` iken
`E02_redundant_few_shot` için `912.3`, `E03_diverse_few_shot` için
`901.3` ve `E04_reversed_order` için `901.3`'tür. Bu, bu özel
çalıştırmada few-shot örneklerin accuracy'ye **hiçbir ek fayda
sağlamadan** prompt token tüketimini yaklaşık %30 artırdığını gösterir —
yani "daha fazla token = daha iyi sınıflandırma" varsayımı burada
doğrulanmamıştır.

### Ceiling Effect (Tavan Etkisi)

Aynı çalıştırmada, `E01_zero_shot`'un zaten %100 accuracy'ye ulaşması bir
**Ceiling Effect** işaretidir: değerlendirme görevi (ve veri seti), seçilen
model için zaten "kolay" olabilir. Bu durumda E02/E03/E04 arasındaki
farkları ölçmek güçleşir, çünkü zaten tavan noktadaki bir metrik daha
yukarı çıkamaz. Böyle bir sonuçla karşılaşıldığında "few-shot işe yaramadı"
sonucuna hemen atlamak yerine, görevin değerlendirme için yeterince zor
olup olmadığı sorgulanmalıdır.

### Format Compliance ile Label Accuracy arasındaki fark — somut bir örnek

LAB01'in kayıtlı bir çalıştırmasında `E01_baseline` (sadece instruction,
Output Contract yok) için `label_accuracy = 0.0` ve `format_compliance_rate
= 0.0` ölçülmüştür; aynı çalıştırmada `E02_context_constraints`,
`E03_output_contract` ve `E04_few_shot` için `label_accuracy = 1.0`'dır.
Ham model yanıtlarına bakıldığında, `E01_baseline`'da model soruyu Türkçe
serbest metinle yanıtlamıştır (örnek: *"**Kategori:** Fatura Hatası /
Fatura ve Ödeme Sorunu..."*) — `billing` gibi beklenen İngilizce anahtar
kelimeyi hiç kullanmamıştır. Parser, yanıtta `VALID_LABELS` kümesindeki
hiçbir terimi bulamadığı için `parsed_label = None` döndürmüş ve bu da
`label_correct = False` olarak işlenmiştir. **Bu, modelin müşteri mesajını
yanlış sınıflandırdığı anlamına gelmez** — model muhtemelen doğru
kategoriyi "anlamış" ama onu beklenen biçimde yazmamıştır. Bu, bir
**Output Contract**'ın neden sadece "biçim" değil, aynı zamanda ölçülebilir
bir accuracy için de gerekli olduğunu gösteren somut bir örnektir.

### Küçük Evaluation Dataset sınırlamaları ve Class Balance

LAB01'de `max_cases = 5`, LAB02'de saldırı başına `LAB02_MAX_CASES = 10`,
LAB03'te `LAB03_MAX_CASES = 10` gibi küçük örnekleme boyutları
kullanılmaktadır. Bu, notebook'ların sınıf içinde hızlı ve düşük maliyetli
çalışabilmesi içindir; ancak tek bir yanlış sınıflandırma bile accuracy'yi
%10-%20 gibi büyük bir oranda değiştirebilir. LAB03'ün değerlendirme veri
seti kategoriler arasında dengelidir (`Class Balance` — her kategoriden eşit
sayıda vaka); LAB02'nin saldırı vakaları ise saldırı türüne göre dengesiz
olabilir (bkz. notebook'un Step 3 uyarısı: küçük `LAB02_MAX_CASES` değerleri
bazı `attack_type`'ları değerlendirme dışında bırakabilir).

### Data Leakage

LAB03, few-shot örnek havuzu (`data/lab03_examples.json`) ile değerlendirme
veri setinin (`data/lab03_evaluation.json`) hiçbir ortak mesaj içermediğini
programatik olarak doğrular. Bu kontrol olmasaydı, modelin yüksek accuracy
elde etmesi "genelleme" değil "ezberleme" (örneği doğrudan görmüş olma)
anlamına gelebilirdi.

### Temperature = 0 ve Determinism

Her üç lab'de `config/settings.toml`'daki `temperature = 0.0` ayarı
kullanılır. Bu, modelin yanıtlarını **daha tutarlı** hale getirir ama
**tam determinizm garanti etmez** — aynı prompt ile yapılan iki ayrı API
çağrısı farklı yanıtlar üretebilir. Bu nedenle tek bir çalıştırmadan elde
edilen sonuçlar kesin bir genel sonuç olarak sunulmamalıdır.

### Latency ölçümlerindeki değişkenlik

LAB02'nin kayıtlı çalıştırmasında `E01_baseline` için `avg_api_latency_ms
≈ 377`, `E03_untrusted_separation` için ise `≈ 4337`'dir — bu fark,
prompt uzunluğundan çok, o anki API/model yüküyle ilgili olabilir.
Latency karşılaştırmaları, tek bir ölçümle kesin bir mühendislik kararına
(örneğin "bu prompt yapısı yavaş" gibi) temel oluşturmamalıdır;
**tekrarlı ölçüm (`repetitions`)** bu değişkenliği azaltmaya yardımcı olur.

### Correlation vs Causation ve Kontrollü Karşılaştırma

LAB02'nin `E02 vs E03` karşılaştırması iyi bir örnektir: kayıtlı
çalıştırmada `E02_naive_concatenation`'ın `model_level_attack_success_rate
= 0.25` (8 saldırı vakasından 2'si başarılı), `E03_untrusted_separation`'ın
ise `0.00` olduğu ölçülmüştür. Bu, `<untrusted_document>` ayrımı +
system-level güven talimatının saldırı başarısını azalttığına işaret
eder — **ama** notebook'un kendisinin de belirttiği gibi, E02→E03 arasında
**iki değişiklik birlikte** uygulanmıştır (belge ayrımı ve güven talimatı).
Bu nedenle "hangi bileşen bu iyileşmeyi tek başına sağladı" sorusu, bu
deney tasarımıyla **cevaplanamaz**; sadece bileşik etkiden söz edilebilir.
Bu, Correlation (iki deney arasında fark görmek) ile Causation (nedeni tek
bir bileşene atfetmek) arasındaki farkı somut olarak gösterir.

## 11. Öğrenci Çalışma Rehberi

Önerilen sıra **LAB01 → LAB02 → LAB03**'tür; her lab bir öncekinin
kavramlarını varsayar.

### LAB01 ile başlayın

- **Önce öğrenin:** System/User mesaj yapısı, prompt bileşenleri
  (instruction, context, constraints, output contract, few-shot
  examples), Output Contract kavramı.
- **Çalıştırın:** `E01_baseline` → `E04_few_shot` sırasıyla dört deneyi,
  `RUN_LIVE = True` ile.
- **Karşılaştırın:** `label_accuracy`, `priority_accuracy`,
  `format_compliance_rate`, token kullanımı.
- **Düşünün:** Output Contract eklenmeden önce ve sonra accuracy nasıl
  değişti? Bu değişim "modelin görevi anlaması" ile mi, yoksa "parser'ın
  yanıtı ayrıştırabilmesi" ile mi ilgili?

### LAB02 ile devam edin

- **Önce öğrenin:** Instruction Hierarchy, Trusted vs Untrusted Content,
  Prompt Injection, `<untrusted_document>`'ın gerçek bir API rolü
  olmadığı.
- **Çalıştırın:** `E01_baseline`, ardından `E02_naive_concatenation` ve
  `E03_untrusted_separation`'ı aynı saldırı vakaları üzerinde; `E04`'ün
  API çağrısı yapmadığını not edin.
- **Karşılaştırın:** `model_level_attack_success_rate` ve
  `system_level_attack_success_rate`'in E02 → E03 → E04 boyunca nasıl
  değiştiğini.
- **Düşünün:** E04'teki deterministik kontrol hangi saldırı türlerini
  yakalayabilir, hangilerini yakalayamaz? Bu sınırlama neden kasıtlıdır?

### LAB03 ile tamamlayın

- **Önce öğrenin:** Zero-Shot vs Few-Shot, In-Context Learning'in
  fine-tuning olmadığı, Example Diversity vs Example Quality ayrımı.
- **Çalıştırın:** `E01_zero_shot` → `E04_reversed_order` sırasıyla;
  Step 6/7'de E01 ile E03'ün ürettiği `messages` dizilerini yan yana
  karşılaştırın.
- **Karşılaştırın:** `label_accuracy`, per-category accuracy,
  `df_order_sensitivity` tablosundaki karar değişim oranı.
- **Düşünün:** Few-shot örnekler accuracy'yi artırdı mı, yoksa sadece
  token maliyetini mi artırdı? Sıralamayı değiştirmek hangi vakalarda
  (varsa) kararı değiştirdi?

Her üç lab'de de, notebook'un sonundaki **Tartışma** / **Discussion**
sorularını yalnızca gerçekten çalıştırdığınız `RUN_LIVE = True`
sonuçlarına bakarak cevaplayın — sonuç uydurmayın.

## 12. Sık Yapılan Hatalar

- **Few-Shot Prompting'i Fine-Tuning ile karıştırmak.** Few-shot örnekler
  model ağırlıklarını değiştirmez; sadece tek bir istek için bağlam
  sağlar. Bir sonraki API çağrısında model, önceki çağrıdaki örnekleri
  "hatırlamaz" — her istek için yeniden gönderilmesi gerekir.
- **Zero-Shot için System Prompt'un kısa olması gerektiğini düşünmek.**
  LAB03'teki `E01_zero_shot`, hiç few-shot example içermez, ama identity,
  instruction, category definitions, classification rules ve output
  contract'ın **tümünü** içerir — "örneksiz" olmak "talimatsız" olmak
  demek değildir.
- **`label_accuracy = %100` sonucunu genel bir başarı garantisi
  saymak.** Küçük bir evaluation dataset üzerinde elde edilen %100, o
  modelin o göreve her koşulda mükemmel uyum sağlayacağı anlamına gelmez
  (bkz. Bölüm 10, Ceiling Effect).
- **Format Compliance ile Label Accuracy'yi aynı metrik saymak.** Bir
  yanıt doğru sınıflandırılmış olabilir ama Output Contract'a uymayabilir
  (ve bunun tersi de mümkün değildir — Output Contract'a uyan her yanıt
  geçerli bir etiket de içermek zorundadır). İkisi, her üç lab'de de
  bilerek birbirinden ayrı hesaplanır.
- **API Error ile Invalid Response'u karıştırmak.** Bir API hatası,
  isteğin kendisinin başarısız olduğu anlamına gelir (`error` alanı
  dolu); bir Invalid Response, isteğin başarılı olduğu ama yanıtın
  tanınan hiçbir kategoriyle eşleşmediği anlamına gelir. İkisi LAB02 ve
  LAB03'te ayrı metrikler olarak (`api_error_rate`,
  `invalid_response_rate`) raporlanır.
- **Prompt Injection savunmalarını kesin bir güvenlik garantisi olarak
  kabul etmek.** LAB02'nin hipotezi, `<untrusted_document>` ayrımının
  saldırı başarısını **azaltabileceğini**, ama tek başına bir güvenlik
  garantisi **sağlamadığını** söyler. Notebook'un kendisi, E04'teki
  Deterministic Validation'ın da yalnızca tek, dar kapsamlı bir ihlali
  yakaladığını açıkça belirtir.
- **Tekrarlı deney yapmadan latency hakkında kesin sonuç çıkarmak.**
  `config/settings.toml`'daki `repetitions` değeri `1` olduğunda, tek bir
  ölçümden "bu prompt yapısı daha hızlı/yavaş" gibi genel bir sonuç
  çıkarmak yanıltıcıdır; latency ağ koşullarına ve o anki API yüküne
  bağlı olarak değişkenlik gösterir.

## 13. Sonuç ve Kazanımlar

LAB01–LAB03'ü tamamlayan bir katılımcı:

- Bir LLM chat completion isteğinin `system` / `user` / `assistant` mesaj
  dizisini kod seviyesinde kurabilir ve bu dizinin gerçek JSON gövdesini
  okuyup yorumlayabilir.
- Prompt bileşenlerini (instruction, context, constraints, output
  contract, few-shot examples) ayrıştırılmış, test edilebilir parçalar
  olarak tasarlayabilir ve bir Output Contract'ın ölçülebilir accuracy
  için neden gerekli olduğunu somut örneklerle açıklayabilir.
- Instruction Hierarchy'nin mesaj rolüne değil **veri kaynağına** dayandığını
  ve `<untrusted_document>` gibi metinsel sınırların tek başına bir
  güvenlik garantisi olmadığını bilir.
- Prompt Injection risklerini gerçek saldırı senaryolarıyla
  değerlendirebilir ve hem model seviyesinde hem uygulama seviyesinde
  alınan önlemlerin **kapsamını abartmadan** yorumlayabilir.
- Zero-Shot ve Few-Shot Prompting'i aynı görev üzerinde karşılaştırabilir;
  Example Selection'ın (redundancy, diversity, ordering) accuracy ve token
  maliyeti üzerindeki etkisini ayrı ayrı değerlendirebilir.
- Label Accuracy, Format Compliance, Invalid Response Rate, API Error
  Rate, Token Usage ve Latency metriklerini birbirinden bağımsız olarak
  okuyabilir ve her birinin ne zaman yanıltıcı olabileceğini (Ceiling
  Effect, Class Balance, Correlation vs Causation, tekrarsız ölçüm gibi)
  fark edebilir.
- Kontrollü bir deney tasarlarken "tek bir değişkeni değiştirip diğer her
  şeyi sabit tutma" ilkesini uygulayabilir ve bir deney tasarımının hangi
  sorulara **cevap veremeyeceğini** (örneğin E02→E03'teki bileşik etki)
  açıkça ifade edebilir.
