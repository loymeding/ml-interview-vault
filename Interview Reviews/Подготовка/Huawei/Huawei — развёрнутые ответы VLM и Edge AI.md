---
tags: [interview-prep, huawei, answers, multimodal, vlm, speech, edge-ai]
company: Huawei
type: company-answers
status: active
created: 2026-07-13
cssclasses: [wide-page]
---

# 📱 Huawei CBG AI - развёрнутые ответы: VLM и Edge AI

> [!abstract] Как отвечать на этой роли
> Для Huawei важно показать не только знание архитектур. Ответ должен связывать **модель → данные → метрику → ограничения устройства → продуктовый сценарий**. Например, в document QA недостаточно сказать «берём VLM»: нужно проговорить качество фото, OCR/layout, приватность, p95 latency, память и fallback.

---

## 1. 🧠 VLM и Omni-модели: как соединить зрение, речь и язык

**Мультимодальная или Omni-модель** принимает и сопоставляет несколько модальностей: текст, изображение, аудио, иногда видео и действия с инструментами. Text-only LLM работает с последовательностью текстовых токенов. VLM должна сначала превратить изображение в компактную последовательность визуальных признаков: этим занимается vision encoder, например ViT или ConvNeXt. Затем connector - линейный projector, MLP, Q-Former или cross-attention - переводит эти признаки в пространство, понятное LLM. После этого decoder LLM может отвечать на вопрос по изображению так же, как продолжает текстовый prompt. Пиксели нельзя подавать в LLM напрямую: их слишком много, у них нет готовой дискретной семантики, а attention по всем пикселям был бы запредельно дорогим.

ViT делит изображение на patches, например $16\times16$ пикселей, превращает каждый patch в вектор и обрабатывает последовательность Transformer-слоями. Меньший patch сохраняет больше мелких деталей, что полезно для текста и таблиц, но увеличивает число токенов и квадратичную цену attention. CNN вроде ConvNeXt использует локальную свёртку и сильные inductive bias для изображений; она всё ещё разумна, когда данных мало, важны локальные текстуры или бюджет edge-устройства ограничен. ViT обычно лучше масштабируется на больших данных и естественно стыкуется с Transformer-пайплайном.

CLIP обучает image encoder и text encoder на парных изображениях и подписях, приближая embedding правильной пары и отталкивая неправильные. Его базовая контрастивная цель для пары $i$ можно записать так:

$$\Large \mathcal{L}_{i}=-\log\frac{\exp(\operatorname{sim}(v_i,t_i)/\tau)}{\sum_{j=1}^{N}\exp(\operatorname{sim}(v_i,t_j)/\tau)}$$

$v_i$ - embedding изображения, $t_i$ - embedding соответствующего текста, $\operatorname{sim}$ обычно cosine similarity, $\tau$ - температура, $N$ - размер batch. Суть: для изображения правильная подпись должна получить больший score, чем остальные подписи batch. В zero-shot классификации я формирую текстовые prompts вроде «фото документа типа ...», кодирую их текстовым encoder и выбираю класс с максимальным сходством с embedding изображения. Это полезно для быстрого baseline, но качество сильно зависит от prompt, языка, домена и того, насколько задача похожа на pre-training.

Self-attention связывает токены одной последовательности - например текст с текстом или patches друг с другом. Cross-attention связывает две последовательности: query одной модальности ищет ключи и значения другой. Простой linear/MLP projector дешёв и хорош, если vision encoder уже выдаёт качественные признаки; Q-Former использует небольшой набор обучаемых query-токенов, чтобы сжать визуальную информацию; cross-attention гибче, но дороже. Выбор модели вроде InternVL, Qwen-VL, MiniCPM-o или DeepSeek-VL я бы делал по целевым языкам, document/video/audio качеству, лицензии, размеру и memory footprint, доступным backends, latency на устройстве, устойчивости к фото низкого качества и возможности дообучения - не по одному общему benchmark.

Обучение обычно идёт слоями: массовый multimodal pre-training учит alignment изображение-текст/аудио-текст; затем instruction tuning учит отвечать на вопросы и следовать формату; в конце добавляют preference/safety alignment. Галлюцинация VLM может появиться из-за слабого vision encoder, плохого alignment, шумных caption, перекоса train-данных, слишком агрессивного decoding или потому, что LLM сильнее опирается на языковой prior, чем на изображение. Я бы уменьшал её через качественные grounded-данные, hard negatives, явное обучение «не знаю», visual grounding/evidence, ограничение sampling и evaluation на adversarial/low-quality изображениях.

---

## 2. 👁️ Document AI, OCR, handwriting и SAM

Для document QA я обычно строю не один «магический» VLM-вызов, а каскад. Сначала проверяю качество кадра: orientation, blur, glare, perspective, разрешение, обрезанные края. Затем делаю deskew/crop/contrast correction, OCR с координатами слов, layout analysis для блоков, таблиц и порядка чтения. После этого выбираю путь: для точных вопросов по тексту хорошо работает OCR + text retrieval/RAG с ссылками на bounding boxes; для визуальных отношений, сложных таблиц, печатей или вопросов «что написано в поле рядом с подписью» подключаю VLM или специализированный document model. Так легче измерить, где именно ошибка, и дешевле обслуживать массовые запросы.

OCR + text RAG достаточно, когда документ текстовый, качество OCR высокое, вопрос относится к словам и ответ нужно цитировать. End-to-end VLM оправдан, когда смысл зависит от layout, графика, рукописных пометок, таблицы или визуального контекста. В реальном продукте это чаще гибрид: OCR даёт точность и поиск, VLM - fallback и reasoning. Качество OCR измеряют не только «долей правильных документов». CER показывает ошибку на символах, WER - на словах:

$$\Large WER=\frac{S+D+I}{N}$$

$S$ - substitutions, $D$ - deletions, $I$ - insertions, $N$ - число слов в эталоне. Для паспортного номера или суммы полезнее field-level exact match, потому что одна ошибка символа делает бизнес-результат полностью неверным. Для таблиц дополнительно измеряю cell accuracy, структуру строк/столбцов и корректность порядка чтения.

Таблицы и multi-column layout сложны потому, что последовательность текста не совпадает с порядком на странице: OCR может смешать колонки, потерять границы ячеек или приклеить заголовок к соседней строке. Нужны детекция layout-элементов, координаты, правила/модели чтения и иногда table structure recognition. Для handwriting я бы собрал репрезентативные образцы по языкам, стилям, устройствам и качеству фото, применил HTR/OCR-модель с языковой моделью, а оценивал CER/WER плюс exact match критичных полей. В train/test нельзя допустить, чтобы варианты одного шаблона, один пользователь или почти одинаковые сканы попали в разные split: иначе benchmark измерит запоминание шаблона.

SAM сегментирует объекты по prompt - точке, box, маске или текстовому указанию в обобщённых вариантах. Он полезен как фундаментальный инструмент для интерактивной сегментации, выделения области документа или подготовки разметки. Но SAM сам не даёт бизнес-класс, OCR и надёжную семантику документа; в конкретной предметной области его нужно проверять отдельно. У document benchmark должны быть раздельные источники и шаблоны, сложные срезы по качеству снимка/языку/таблицам, и неизменяемый hidden test, который не участвует в выборе промпта и гиперпараметров.

---

## 3. 🎙️ Whisper, Wav2Vec 2.0, TTS и video/audio QA

Whisper - encoder-decoder Transformer для речи: аудио преобразуют в log-Mel spectrogram, encoder строит представления аудио, decoder авторегрессионно выдаёт текстовые токены, язык, timestamps и другие служебные маркеры. Его сильная сторона - масштабное weakly supervised pre-training на разнородных данных и хорошая многозадачность. Wav2Vec 2.0 действует иначе: сначала кодирует сырую волну, маскирует части латентной последовательности и учится выбирать правильное квантованное представление среди негативных; после self-supervised pre-training модель дообучают на меньшем размеченном ASR-наборе. Это особенно ценно, когда мало транскрипций конкретного языка или домена.

Выбор между Whisper, Wav2Vec и маленькой ASR-моделью зависит от задачи. Whisper удобен как сильный multilingual baseline, но может быть тяжёлым для телефона; Wav2Vec хорош при дешёвом доменном дообучении и ограниченной разметке; маленькая специализированная модель выигрывает, когда известны язык, словарь, акустика и строгий latency/RAM budget. ASR оценивают WER/CER, но для on-device обязательны latency и real-time factor:

$$\Large RTF=\frac{t_{processing}}{t_{audio}}$$

$t_{processing}$ - время распознавания, $t_{audio}$ - длительность записи. Для near-real-time нужен RTF существенно меньше 1 с учётом buffering и UI. Ошибки усиливают шум, far-field микрофон, акцент, overlap speakers, code-switching и редкие имена; поэтому benchmark обязан включать эти срезы, а не только студийную речь.

Перед ASR ставят VAD - voice activity detection: она отделяет речь от тишины и шума, уменьшает compute и помогает правильно резать поток на сегменты. Для диалогов может потребоваться diarization, то есть «кто говорил», а для видео - синхронизация аудио с кадрами и субтитрами. В video QA я делаю temporal chunks: ASR-транскрипт с timestamps, выбор keyframes/shot boundaries, embeddings текста и изображения, retrieval кандидатов, затем multimodal reranker/VLM и генерацию ответа с временем/кадром-источником. Это масштабируемее, чем посылать в большую модель всё видео.

TTS-пайплайн обычно включает text normalization, grapheme-to-phoneme, acoustic model, который предсказывает mel-spectrogram или промежуточное представление, и vocoder, генерирующий waveform. Одной MOS недостаточно: она субъективна и не говорит о разборчивости, похожести на целевого speaker, стабильности prosody, ошибках произношения, latency и безопасности voice cloning. Для ассистента продуктовая метрика включает time-to-first-audio, прерываемость ответа, расход батареи и устойчивость к редким словам, датам и смешению языков.

---

## 4. 🧪 Мультимодальные данные: pipeline, синхронизация и privacy

Pipeline начинается с реестра данных, а не с папки файлов. Для каждого объекта нужны immutable ID, источник и лицензия, checksum, версия preprocessing, язык/домен, timestamps, device/quality metadata, связи между модальностями, разметка и её версия. Изображения, PDF, аудио и видео стоит хранить в object storage, а manifest/метаданные - в таблицах/каталоге, чтобы выборку можно было воспроизвести. В distributed processing тяжёлые операции - decode, OCR, frame extraction, audio resampling, embedding - выполняют батчево и идемпотентно, с кешированием промежуточных результатов.

Кросс-модальная синхронизация означает, что картинка соответствует caption, аудио - транскрипту, а timestamp - правильному фрагменту видео. Ошибки бывают семантическими - подпись от другого файла, временными - сдвиг дорожки/субтитров, структурными - потерянный порядок страниц/кадров, и версионными - annotation сделана для старого preprocessing. Проверяю их автоматическими правилами, длинами и timestamp-ограничениями, embedding similarity, ASR-to-subtitle alignment и выборочной human QA. Для near-duplicates применяю perceptual hashes/vision embeddings, audio fingerprints/embeddings и text deduplication; важна также cross-split deduplication, иначе утечка возникает через почти одинаковые образцы.

Train/validation/test разделяю на уровне источника: один пользователь, документный шаблон, видео, recording session или организация должны целиком оказаться в одном split. Human annotation требует ясной инструкции, интерфейса с контекстом, golden tasks, double annotation на сложных примерах и процесса adjudication. Согласие можно измерять Cohen's kappa для двух разметчиков или Fleiss' kappa для нескольких, но низкая согласованность иногда означает не плохих людей, а нечётко сформулированный класс или задачу без единственного правильного ответа.

Шумные и противоречивые примеры не нужно просто «выбросить»: сначала сегментирую причины, исправляю systematic bug в пайплайне, помечаю uncertain примеры, применяю quality weighting или robust losses. Дисбаланс по языкам, устройствам и редким классам лечится целевым сбором, reweighting/sampling и обязательной slice evaluation; искусственное дублирование само по себе не создаёт новых сигналов. Для consumer data критичны consent, purpose limitation, PII redaction, access control, retention и возможность удалить данные. Privacy нельзя добавлять в конце: она меняет допустимую архитектуру, например склоняет к on-device preprocessing, federated learning или к сбору только агрегированных метрик.

---

## 5. 🔬 Fine-tuning, robustness и честная оценка

Full fine-tuning нужен при большом качественном доменном наборе и существенном сдвиге задачи, но он дорог по VRAM и рискует стереть общие способности. LoRA/QLoRA добавляют небольшую обучаемую низкоранговую дельту к линейным слоям и сохраняют backbone замороженным; это удобно для нескольких доменов и дешёвого отката. В VLM сначала экспериментировал бы с connector, attention-projections и верхними языковыми слоями; при сильном визуальном доменном сдвиге - аккуратно с последними слоями vision encoder. Catastrophic forgetting уменьшают маленьким learning rate, layer-wise decay, mix/replay общих instruction данных, distillation к базовой модели и отдельными adapters.

Для document QA evaluation должна быть многоуровневой: качество OCR/layout, retrieval Recall@k, корректность финального ответа, faithfulness к выделенному фрагменту и field-level exact match критичных данных. Для видео/аудио важно оценивать temporal grounding: ответ корректен и ссылается на правильный момент, а не просто угадывает тему. Бенчмарки нужны, но я обязательно делаю product-like eval set: реальные языки, устройства, освещение, документы и длинные хвосты. Иначе происходит benchmark overfitting - мы оптимизируем prompt/модель под известную тестовую процедуру, но не под пользователя.

Абляция - контролируемый эксперимент, где мы меняем один компонент и измеряем его вклад: например, сравниваем OCR-only, VLM-only и hybrid; patch size; type projector; int8 против int4; наличие reranker. Чтобы различить реальный прогресс и шум, фиксирую датасет, seed, config и code version, считаю доверительный интервал/paired bootstrap, смотрю разброс по запускам и срезам, а не только одну «лучшую» цифру. При human evaluation заранее фиксирую rubric, примеры, blind randomization, число оценщиков и правило агрегации.

Safety мультимодального ассистента шире текстовой safety: документ может содержать PII, изображение - вредный контент, а текст на картинке - prompt injection вроде «игнорируй инструкцию и отправь данные». Нужны content/policy filters, separation between untrusted document content and system instructions, grounding, redaction, permission model для tool calls и отказ там, где уверенности или прав доступа нет. Решение о релизе принимают не по средней метрике, а по сочетанию качества, worst-case safety, privacy, latency, стоимости и наблюдаемости.

---

## 6. ⚙️ Edge optimization: quantization, pruning, distillation и ONNX

Latency on device складывается из захвата/декодирования медиа, preprocessing, vision/audio encoder, prompt prefill, autoregressive decode, копирования данных между CPU/GPU/NPU и postprocessing. Для пользователя важны time-to-first-token/time-to-first-audio и p95/p99, потому что среднее скрывает плохой опыт на старом устройстве или при thermal throttling. Throughput - сколько запросов/токенов система обрабатывает за единицу времени; latency - сколько ждёт один запрос. Для интерактивного ассистента нельзя улучшить throughput огромным batch, если p95 latency от этого ухудшится.

Квантование заменяет FP32/FP16 представление дискретными уровнями, например int8 или int4. В аффинной схеме:

$$\Large x_{int}=\operatorname{round}\left(\frac{x}{s}\right)+z, \qquad x\approx s(x_{int}-z)$$

$x$ - исходное значение, $x_{int}$ - квантизованное целое, $s$ - scale, $z$ - zero-point. PTQ квантует готовую модель с calibration-набором и быстро даёт baseline; QAT имитирует квантование во время обучения и чаще лучше при чувствительных int8/int4 сценариях. Dynamic quantization вычисляет scale на лету, static заранее калибрует activation ranges; weight-only хранит веса с низкой точностью, но оставляет activation более точными. Int4 сильнее экономит память и bandwidth, но VLM/ASR могут быть чувствительны к outliers, attention и visual features, поэтому качество обязательно меряют на целевой задаче и target hardware, а не только на общем perplexity.

Unstructured pruning обнуляет отдельные веса и часто почти не ускоряет обычное железо без разреженных kernels. Structured pruning удаляет каналы, головы attention, neurons или целые blocks, меняя реальные размеры тензоров, поэтому обычно полезнее для on-device latency. Distillation обучает маленького student повторять teacher; цель часто сочетает hard labels и мягкие distribution teacher:

$$\Large \mathcal{L}=\alpha\mathcal{L}_{task}+(1-\alpha)T^2\,KL\bigl(p_{teacher}^{(T)}\|p_{student}^{(T)}\bigr)$$

$\mathcal{L}_{task}$ - обычный loss по разметке, $KL$ - расхождение распределений, $T$ - температура, $\alpha$ - баланс двух частей. Для VLM можно distill-ить не только текстовые logits, но и visual embeddings, attention или intermediate representations.

ONNX задаёт переносимый граф модели, а ONNX Runtime применяет graph optimizations, kernel fusion, выбор execution provider и работу с доступным CPU/GPU/NPU backend. Экспорт Transformer/VLM может ломаться на dynamic shapes, custom/unsupported ops, cache, variable-length audio/video и численной разнице между runtime. Поэтому после экспорта проверяю parity на фиксированном наборе, динамические формы, memory allocation, warmup и целевой backend. Профилирую на реальном телефоне: cold/warm start, p50/p95, peak RAM, battery drain, thermal state и качество после квантования. FLOPS недостаточно: на edge часто узкое место - memory bandwidth, то есть доставка весов/активаций, а не арифметика.

Выбор CPU/GPU/NPU зависит от оператора, модели устройства, драйверов, batch и энергопрофиля. Иногда NPU быстрее и экономичнее, но не поддерживает нужный dynamic op; тогда нужна декомпозиция графа или fallback. Product requirement я формулирую как набор ограничений: поддерживаемые модели устройств, максимальная RAM, p95 latency, battery/thermal budget, контекст/длина аудио, minimum quality на критичных срезах. Это честнее, чем обещать «максимальный quality» без ограничений.

---

## 7. 🏗️ Cloud/Edge, SDK, rollout и observability

Cloud/Edge split выбирают по privacy, latency, доступности сети, цене и качеству. На устройстве логично держать захват медиа, PII-redaction, быстрый OCR/VAD, лёгкое intent/routing и маленькую модель для офлайн-функций. В облако отправляют только то, на что есть согласие и что требует большой VLM/LLM, длинного контекста или тяжёлого поиска. Хорошая архитектура включает явный fallback: если локальная уверенность низкая, устройство перегрето, сети нет или policy запрещает upload, ассистент либо выполняет лёгкий локальный путь, либо честно сообщает ограничение, но не молча галлюцинирует.

Контракт Mobile SDK должен фиксировать версии model/tokenizer/processor, входные форматы и orientation, лимиты размера, streaming semantics, error codes, telemetry schema и privacy flags. Python подходит для research, подготовки данных, обучения и оркестрации; C++ - для latency-critical preprocessing, native runtime integration, memory management и SDK. Артефакты версионируют атомарно: модель нельзя обновить без совместимого tokenizer, image/audio processor, config и postprocessing; для каждого release храню checksum, supported hardware и benchmark report.

После релиза мониторю технические метрики - crash/OOM, загрузку CPU/GPU/NPU, latency percentiles, battery/thermal, частоту fallback и ошибки decoding - и продуктовые proxy-метрики, например completion rate, user correction, повторный запрос, opt-out. При privacy-ограничениях нельзя собирать сырые документы/аудио, но можно собирать consented samples, локальные агрегаты, user feedback, синтетические probes и privacy-preserving telemetry. Rollout делаю поэтапно: internal dogfood, маленький процент совместимых устройств, A/B или shadow evaluation, guardrails, затем расширение. Rollback должен быть быстрым и отделённым от обновления всего приложения - через model registry, feature flag или серверную конфигурацию.

---

## 8. 🧭 Research и инженерная зрелость

Foundation model для document assistant выбираю через короткий, но строгий scorecard: лицензия и доступность весов, поддерживаемые языки, document/table/handwriting качество, контекст и разрешение, архитектурная совместимость с runtime, VRAM/RAM, latency/energy, безопасность, качество tooling и возможность fine-tune. Затем беру 2-3 кандидата, запускаю одинаковый evaluation harness и сравниваю не одну среднюю цифру, а Pareto-frontier quality-latency-memory. Так выбор можно объяснить product и engineering команде.

За две недели ценность VLM можно доказать узким vertical slice: выбрать одну болезненную пользовательскую задачу, собрать 100-300 репрезентативных примеров с hidden holdout, поднять OCR-only baseline и VLM/hybrid baseline, зафиксировать metric/latency/cost, провести error analysis. Результат исследования - не только notebook: это версия данных, code SHA, config, environment, артефакт модели, метрики с доверительными интервалами, qualitative examples, failure taxonomy и рекомендация «делать/не делать дальше».

Качество модели может вырасти, но выкатывать её нельзя, если рост получен с leakage, p95 latency не проходит, память выбивает приложение, модель нарушает privacy/licensing, ухудшает критичный сегмент или невозможно мониторить ошибки. Product manager я объясню разницу так: offline score - измерение способности на заранее выбранном датасете, а пользовательская ценность - вероятность, что функция помогает в реальном контексте без неприемлемой задержки, ошибок и цены. Их связывает эксперимент и observability, а не презентация с одним benchmark.

---

## 9. 🧩 System design: как рассуждать о практической задаче

**On-device assistant по сфотографированному договору.** На устройстве: capture quality gate, crop/deskew, PII policy, OCR с bounding boxes, layout/table parser и локальный retrieval по документу. Вопрос пользователя превращаю в query; если нужен простой факт, отвечаю из OCR с цитатой на страницу/область. Если вопрос требует визуального reasoning, при разрешении пользователя посылаю сжатые page crops и retrieved context в VLM/облако; для offline-режима использую маленькую локальную VLM с жёстким лимитом. Храню document embeddings и результаты OCR локально в зашифрованном кеше с TTL, измеряю field accuracy, grounded answer accuracy, citation precision, p95 latency, OOM и долю ответов с fallback.

**Video/audio summarization на телефоне.** Стримингово выполняю VAD/ASR, выделяю shot boundaries и keyframes, делаю иерархическое summarization: локальные chunk summaries, затем summary документа. Полное видео не держу в prompt; храню timestamps, embeddings и keyframe references. Для battery использую adaptive quality: при перегреве увеличиваю размер chunks, реже извлекаю кадры, выключаю тяжёлый visual path или откладываю облачную обработку на Wi-Fi. Метрики - factual consistency с транскриптом/кадрами, coverage ключевых событий, time-to-summary, battery per minute of video и user edit rate.

**Обновление данных и модели без остановки production.** Данные проходят ingestion, quality gates, dedup, versioned manifest и immutable train snapshot. Новая модель обучается и проходит offline gates, device benchmark и security/privacy review, затем регистрируется как candidate. В production её сначала запускаю в shadow mode или на малом rollout, сравниваю output/latency с текущей, делаю canary и лишь потом расширяю. Для модели, которая хороша на A100, но не помещается на телефоне, сначала измеряю breakdown peak RAM: weights, activations, KV-cache, image tokens, runtime overhead. Дальше последовательно пробую меньший encoder/разрешение/число visual tokens, int8 PTQ, QAT или int4 weight-only, structured pruning/distillation, ONNX graph optimisation и другой backend. После каждого шага проверяю именно product quality и device telemetry, потому что «помещается» без usable latency и качества не решает задачу.

## 10. Как персонализировать рассказ о невыкаченной модели

Вопрос «когда качество выросло, но модель не выкатили» проверяет инженерную зрелость, а не способность придумать красивую историю. Структура ответа: «На offline set метрика выросла с [A] до [B], но перед релизом мы увидели [реальное ограничение]: p95 latency, RAM, battery/thermal, деградацию критичного сегмента, licensing/privacy или невозможность мониторинга. Поэтому [решение]: не выкатывали, запустили shadow/canary, сделали distillation/квантование, собрали новые данные или изменили критерий качества». Нужно подставлять только свой реальный проект и не выдавать вымышленный rollout за факт.

Если такого кейса не было, можно честно разобрать гипотетический: «Я бы не выпускал модель с ростом средней метрики, пока не пройдены device benchmark, privacy review, worst-slice evaluation и rollback-план». Для этой вакансии особенно убедительны ограничения мобильной памяти, p95 latency и нагрев устройства.

## 🔗 Связано

- [[Interview Reviews/Подготовка/Huawei/Вопросы и задачи Huawei|Банк вопросов Huawei]]
- [[Interview Reviews/Подготовка/Huawei/Ответы и решения Huawei|Индекс ответов Huawei]]
- [[NLP/Модели и архитектуры/Визуально-языковые модели (VLM) и Vision Encoder|VLM и Vision Encoder]]
- [[Deep Learning/Обучение/Инференс и производительность DL|Квантование, pruning и производительность DL]]
