# 2F-AGRO — Documento de Design

**Global Solution · FIAP · 1º semestre 2026 · 3ES — Engenharia de Software**

> **2F-AGRO**
> *Tecnologia espacial acessível pra quem alimenta o Brasil.*
>
> Plataforma que combina dados de satélite + sensores de baixo custo + IA + visão computacional pra que **pequenos agricultores, cooperativas familiares e assentamentos** tomem decisões agronômicas com a mesma qualidade que grandes fazendas — sem precisar entender de tecnologia.

---

## 1. Visão e proposta de valor

### O problema

A agricultura familiar é responsável por **70% do alimento na mesa do brasileiro** e ocupa **77% dos estabelecimentos rurais** (IBGE). Apesar disso, **toda a tecnologia agrícola disponível hoje atende o grande agro**: Climate FieldView (Bayer), Strider, Solinftec, Aegro — todas focadas em fazendas grandes com infraestrutura, conhecimento técnico e capital.

O pequeno produtor — quem mais precisaria de ajuda pra prever uma geada, identificar uma praga ou planejar o plantio — é **deserto tecnológico**.

### A solução

2F-AGRO é uma plataforma que aproxima esse pequeno produtor da tecnologia espacial usando **3 camadas integradas**:

1. **Camada orbital** — dados simulados de satélite (NDVI = saúde da vegetação, umidade do solo, cobertura de nuvens)
2. **Camada edge** — estação meteorológica autônoma de baixo custo na propriedade (Raspberry Pi solar)
3. **Camada bolso** — app mobile acessível no celular do agricultor + visão computacional pela câmera (foto da folha → detecta praga)

Sobre as três, um pipeline de Machine Learning treinado no dataset oficial da FIAP prevê **risco de perda de safra**, agrupa propriedades por **perfil agronômico**, e dispara recomendações simples e diretas pro agricultor.

### Stakeholders

| Quem | O que ganha |
|---|---|
| 🧑‍🌾 **Pequeno agricultor / familiar** | Recomendações claras, alertas em tempo, evita prejuízo |
| 👥 **Cooperativa / assentamento** | Visão agregada da região, ação coordenada, defesa coletiva |
| 🏛️ **EMATER, MAPA, ASA** | Apoio extensionista escalado por tecnologia |
| 🏦 **Bancos / Pronaf** | Reduz risco de crédito rural |
| 🌱 **ONGs (CONTAG, MST)** | Ferramenta de inclusão tecnológica do campo |

### ODS atendidos (5 de uma vez)

- **ODS 1** — Erradicação da pobreza (rural)
- **ODS 2** — Fome zero e agricultura sustentável **← primário**
- **ODS 8** — Trabalho decente e crescimento econômico
- **ODS 13** — Ação contra a mudança climática
- **ODS 15** — Vida terrestre (uso sustentável do solo)

### Conexão com o briefing oficial

- Briefing pg. 3 cita literalmente *"Monitoramento agrícola com dados de satélite para aumento de produtividade"* e *"Aplicações de telemedicina e saúde em regiões isoladas"* — 2F-AGRO é a primeira, e o segundo segue a mesma filosofia de inclusão remota
- Enunciado do ML cita *"plataforma de apoio à defesa civil, agricultura e gestão ambiental"* — agricultura é o foco
- Enunciado do OS cita *"como um sistema operacional otimizado para baixo consumo de energia em um satélite pode ser usado em dispositivos IoT para monitoramento agrícola em áreas remotas sem infraestrutura de energia confiável"* — **a banca literalmente sugeriu esse cenário pra OS**

---

## 2. Pilares de acessibilidade (o diferencial)

```
1. 📱 ACESSO RADICAL NO APP
   • Funciona em celular Android básico (Android 7+)
   • Offline-first: sincroniza quando pegar sinal
   • Modo "baixo dado": consumo < 5 MB/mês
   • Interface com cores semafóricas (🟢🟡🔴), ícones grandes
   • Áudio em português ("Ouvir alerta") pra baixa alfabetização

2. 🧠 IA INVISÍVEL
   • Agricultor não vê "ML" — vê: "Sua roça tem 85% de chance de
     perder produtividade em 2 semanas. Motivo: seca + vegetação fraca."
   • Recomendação direta: "Irrigue lavoura 3 hoje" / "Adie o plantio 4 dias"
   • Foto da folha → IA identifica praga/doença em 1 segundo

3. 🤝 PODER COLETIVO
   • Dados agregados da cooperativa = mapa de risco regional compartilhado
   • Compartilhamento opt-in entre vizinhos (LGPD respeitado)
   • Sincroniza com EMATER pra ação coordenada
```

---

## 3. Arquitetura geral (visão SOA)

```
┌─────────────────────────────────────────────────────────────────────┐
│                       USUÁRIOS                                      │
│  🧑‍🌾 Agricultor   👥 Cooperativa   🏛️ EMATER/MAPA                   │
└──────────────┬───────────────────┬────────────────────┬─────────────┘
               │                   │                    │
       ┌───────▼──────┐    ┌───────▼──────┐    ┌────────▼──────┐
       │  📱 APP RN   │    │  🌐 WEB DASH │    │  📡 API EXT.  │
       │  (Expo)      │    │  (React)     │    │  (EMATER/MAPA)│
       │  ★ ACESSÍVEL │    │              │    │               │
       └───────┬──────┘    └───────┬──────┘    └────────┬──────┘
               │                   │                    │
   ════════════╪═══════════════════╪════════════════════╪══════════
               ▼                   ▼                    ▼
   ┌───────────────────────────────────────────────────────────────┐
   │       🔐 API GATEWAY + AUTH (OAuth2 / JWT) — C# .NET          │
   └─────────────────────────────┬─────────────────────────────────┘
                                 │
   ┌────────────┬────────────────┼────────────────┬──────────────┐
   │            │                │                │              │
   ▼            ▼                ▼                ▼              ▼
┌───────┐  ┌──────────┐  ┌─────────────┐  ┌────────────┐  ┌──────────┐
│Serviço│  │ Serviço  │  │  Serviço    │  │  Serviço   │  │ Serviço  │
│Alertas│  │Notif.    │  │  ML Agro    │  │  Ingestão  │  │ Auth +   │
│Agro   │  │(Push/    │  │  (Python +  │  │  Sat/Edge  │  │ Audit    │
│ (C#   │  │ Email)   │  │  scikit-l)  │  │  IoT       │  │ (LGPD)   │
│ .NET) │  │          │  │             │  │            │  │          │
└───┬───┘  └────┬─────┘  └──────┬──────┘  └─────┬──────┘  └────┬─────┘
    │           │               │               │              │
    └───────────┴───────┬───────┴───────────────┴──────────────┘
                        ▼
        ┌──────────────────────────────────┐
        │  PostgreSQL + PostGIS  +  S3     │
        └──────────────────────────────────┘
                        │
       ┌────────────────┼─────────────────┐
       ▼                ▼                 ▼
   🛰️ SAT SIM       📡 ESTAÇÃO       📷 CÂMERA
   (CSV FIAP)       NA FAZENDA       DO AGRICULTOR
   NDVI, nuvens     (Rasp Pi+OS)     (foto da folha)
```

### Decomposição em microsserviços (arquitetura macro)

| Serviço | Responsabilidade | Tecnologia | Matéria principal |
|---|---|---|---|
| **Serviço SOA (Java)** | REST `Propriedade` + SOAP cadastro rural + integração NASA/CPTEC | Java 17 + Spring Boot + Spring-WS + H2 | **SOA** (ver § 4.7) |
| **API Gateway** | Roteamento, auth, rate limit | C# .NET 8 Web API | C# |
| **Serviço Alertas Agro** | Regras agronômicas, persistência | C# .NET 8 + EF Core | **C#** |
| **Serviço Notificação** | Push (Expo), email | C# .NET + fila | C# / infra |
| **Serviço ML Agro** | Inferência (Reg.Linear + Logística + Árvore + K-Means) | Python FastAPI + scikit-learn | **ML** |
| **Serviço Visão Praga** | YOLO/CNN em folhas → identifica praga | Python + YOLOv8 | **IoT / CV** |
| **Serviço Ingestão** | Recebe CSV satelital e dados da estação | Python + pandas | ML |
| **Serviço Auth** | OAuth2 + JWT + audit | C# Identity | Cyber |
| **App Mobile** | Cliente principal — agricultor | React Native + Expo | **Mobile** |
| **Web Dashboard** | Painel cooperativa / EMATER | React + Leaflet | (bônus) |
| **Estação Edge** | Coleta sensores na fazenda | Python + Raspbian custom | **OS + IoT** |

---

## 4. Componentes detalhados por matéria

### 4.1 ML — *Risco Agronômico Inteligente* (10 pts)

**Entrega:** Notebook Jupyter executado + Relatório PDF (até 6 pgs) + CSV usado.

**Reinterpretação do dataset pro contexto agro:**

| Coluna do CSV | Significado agronômico |
|---|---|
| `indice_vegetacao_ndvi` | 🌱 Saúde da vegetação (próximo de 1 = saudável; baixo = stress) |
| `umidade_solo_pct` | 💧 Disponibilidade hídrica (pra raízes) |
| `chuva_prevista_mm` | 🌧️ Demanda/excesso hídrico próximo |
| `temperatura_media_c` | 🌡️ Stress térmico (>35°C ou <10°C = perda) |
| `vento_kmh` | 💨 Risco de danos físicos / dispersão de pragas |
| `cobertura_nuvens_pct` | ☁️ Disponibilidade solar pra fotossíntese |
| `inclinacao_terreno_graus` | ⛰️ Erosão e perda de solo |
| `historico_eventos_5anos` | 📊 Vulnerabilidade histórica |
| `densidade_populacional_km2` | 👥 Pressão de uso da terra |
| `proximidade_area_urbana_km` | 🏙️ Acesso a mercado / poluição |
| **`indice_impacto`** | **Índice de perda esperada da safra (target regressão)** |
| **`risco_alto`** | **Alto risco de perda (target classificação)** |

**Pipeline obrigatório (6 etapas):**

```python
# Etapa 1 — Carregar e entender
df = pd.read_csv('dataset_space_risk_global_solution.csv')
# 320 propriedades/regiões agrícolas, 14 colunas

# Etapa 2 — EDA + correlação
sns.heatmap(df.drop('regiao_id', axis=1).corr(), annot=True)
# Espera-se: NDVI tem correlação NEGATIVA forte com risco_alto
#            (NDVI baixo = vegetação fraca = alto risco)

# Etapa 3 — Regressão Linear (prever indice_impacto)
X = df.drop(['regiao_id', 'indice_impacto', 'risco_alto'], axis=1)
y = df['indice_impacto']  # impacto na safra
# LinearRegression + R² + MAE

# Etapa 4 — Regressão Logística (prever risco_alto)
# IMPORTANTE: não usar indice_impacto (vazamento)
pipe = Pipeline([('scaler', StandardScaler()),
                 ('lr', LogisticRegression())])
# stratify=y + predict_proba + matriz confusão

# Etapa 5 — Árvore de Decisão (max_depth=4)
dt = DecisionTreeClassifier(max_depth=4, random_state=42)
# export_text → regras que o gestor da cooperativa entende

# Etapa 6 — K-Means (k=4) sem usar risco_alto
features_cluster = ['temperatura_media_c', 'umidade_solo_pct',
                    'indice_vegetacao_ndvi', 'chuva_prevista_mm',
                    'inclinacao_terreno_graus']
# Clusters interpretados:
```

**Clusters K-Means (perfis agronômicos):**
- **Cluster 0 — Propriedades estáveis** (NDVI alto, umidade adequada) → safra segura
- **Cluster 1 — Zonas secas** (temp alta, umidade baixa, NDVI baixo) → risco de seca/incêndio
- **Cluster 2 — Áreas chuvosas/encharcadas** (chuva alta, umidade saturada) → risco de alagamento/fungos
- **Cluster 3 — Encostas vulneráveis** (inclinação alta + chuva) → risco de erosão/deslizamento

**Perguntas obrigatórias respondidas no relatório:**

1. *R² alto → variáveis ambientais explicam bem o impacto na safra* (sim — NDVI, umidade e chuva são preditores diretos de produtividade)
2. *Probabilidade próxima de 1* → alta confiança que a propriedade vai sofrer perda significativa
3. *Falso positivo ou falso negativo é mais perigoso?* → **Falso negativo**. Classificar uma propriedade de alto risco como baixo deixa o agricultor sem aviso e ele pode perder safra inteira (R$ 10-50k de prejuízo direto pra família). Falso positivo "só" gera mobilização desnecessária. Estratégia: ajustar threshold pra favorecer recall sobre precisão.
4. *Árvore mais explicável?* → Sim — gerente de cooperativa entende "Se NDVI < 0.3 e chuva > 80mm → ALTO RISCO" sem precisar saber estatística
5. *Modelo mais útil?* → **Logística** (probabilidade calibrada pro alerta) + **Árvore** (regras pro gestor); K-Means complementa pra segmentação regional

**ODS conectados na conclusão:** ODS 2 (Fome Zero) + ODS 1 (Pobreza) + ODS 13 (Clima).

### 4.2 IoT — Visão Computacional de Pragas (100 pts)

**Entrega:** Vídeo demo + script Python + repo Git com README + requirements.txt

**Solução:** Módulo "Olho na Folha" — script Python que roda via webcam capturando folhas de plantas e classifica praga/doença em tempo real.

**Datasets pra treinar/fine-tune (open):**
- [PlantVillage](https://www.kaggle.com/datasets/abdallahalidev/plantvillage-dataset) — 38 classes, ~50k imagens, doenças de tomate/milho/maçã/café/uva
- [PlantDoc](https://github.com/pratikkayal/PlantDoc-Dataset) — fotos reais de campo, mais ruidoso (mais realista)

**Stack:** Python 3.10 + OpenCV + Ultralytics YOLOv8 + Pillow

**Doenças/pragas pra detectar (sugestão):**
- 🍅 Mancha bacteriana do tomate
- 🌽 Lagarta-do-cartucho (Spodoptera frugiperda)
- ☕ Ferrugem do café (Hemileia vastatrix)
- 🥬 Mofo branco / Oídio
- 🍇 Míldio da videira

```python
import cv2
from ultralytics import YOLO
from datetime import datetime
import requests, sys

# Modelo: YOLO custom fine-tuned em PlantVillage
model = YOLO('models/2fagro-folha-v1.pt')

API_URL = 'http://localhost:5000/api/diagnostico'

cap = cv2.VideoCapture(0)
if not cap.isOpened():
    raise RuntimeError("Webcam indisponível")

try:
    while True:
        ret, frame = cap.read()
        if not ret:
            print("Frame perdido — continuando...")
            continue

        results = model(frame, verbose=False)
        annotated = results[0].plot()

        # Mostrar FPS na tela (rubrica)
        fps = cap.get(cv2.CAP_PROP_FPS)
        cv2.putText(annotated, f"FPS: {fps:.1f}",
                    (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.8,
                    (0, 255, 0), 2)

        # Detecção com confiança alta → registra
        for det in results[0].boxes.data.tolist():
            x1, y1, x2, y2, conf, cls = det
            if conf > 0.7:
                requests.post(API_URL, json={
                    'praga': model.names[int(cls)],
                    'confianca': round(conf, 2),
                    'timestamp': datetime.utcnow().isoformat()
                })

        cv2.imshow('2F-AGRO — Olho na Folha', annotated)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

except KeyboardInterrupt:
    print("Encerrando...")
except Exception as e:
    print(f"Erro no stream: {e}", file=sys.stderr)
finally:
    cap.release()
    cv2.destroyAllWindows()
```

**Pontos da rubrica que essa solução marca:**
- ✅ Tratamento de exceções no stream (rubrica obrigatória)
- ✅ FPS visível na tela
- ✅ Robustez a iluminação (YOLO é robusto + augmentation no treino)
- ✅ requirements.txt + README + diagrama da pipeline

### 4.3 OS — Estação Meteo Solar na Fazenda (10 pts)

**Cenário escolhido:** Estação meteorológica autônoma instalada em propriedade rural no semiárido nordestino, alimentando o 2F-AGRO com dados de sensores em tempo real, **sem energia elétrica da rede**.

**Encaixe perfeito com o briefing:** o PDF do OS sugere literalmente esse cenário (*"monitoramento agrícola em áreas remotas sem infraestrutura de energia confiável"*).

**Arquitetura de hardware simulada:**
- **Compute:** Raspberry Pi 4 (4GB RAM, eMMC 32GB)
- **Sensores:** DHT22 (temp+umidade ar), capacitivo de solo, anemômetro reed switch, pluviômetro tipo báscula, BME280 (pressão)
- **Energia:** painel solar 20W + bateria LiFePO4 12.8V/10Ah + MPPT
- **Conectividade:** LoRaWAN (gateway na sede da cooperativa) com fallback GSM/4G
- **Custo total estimado:** ~R$ 600-800 (acessível pra cooperativa)

**OS base:** **Raspbian Lite 64-bit** com kernel `PREEMPT-RT` patch, em vez de RTOS puro (FreeRTOS/Zephyr).

**Justificativa:** RTOS puro seria over-engineering pra latência aceitável (~segundos). Linux com PREEMPT-RT entrega determinismo suficiente + ecossistema Python/sensores/MQTT pronto.

**3 estratégias de tuning (rubrica pede 3):**

#### 1. Gerenciamento de Memória (evitar OOM)
- `vm.swappiness=10` — minimiza swap (eMMC tem desgaste por escrita)
- **cgroups v2** limitando processo `data-collector` a 256MB e `cv-edge` a 512MB
- **earlyoom daemon** mata processos não-críticos antes do kernel travar
- `kernel.min_free_kbytes=131072` — reserva 128MB pro kernel
- Watchdog hardware do BCM2711 reseta em deadlock

#### 2. CPU Scheduling (prioridade pra crítico)
- Processo de **telemetria** roda com `SCHED_FIFO` prio 99 (real-time)
- `chrt -f 99 python telemetria.py` no boot via systemd
- Processos de coleta secundária em `SCHED_OTHER` com `nice +10`
- **Afinidade de CPU** (`taskset`) dedica core 0 ao crítico, cores 1-3 ao resto
- `cpufreq` em modo `ondemand` pra economia quando inativo

#### 3. I/O e Armazenamento (durabilidade)
- Sistema de arquivos **F2FS** (Flash-Friendly File System) — reduz desgaste
- `commit=60` no mount (reduz fsync writes)
- **Ring buffer em RAM** (`tmpfs`) pra dados quentes → flush em batch a cada 60s
- **Write barriers** habilitados pra integridade contra falha de energia
- `logrotate` agressivo (24h + compressão lzma) pra logs
- **fsync seletivo** apenas em commits críticos

### Conexão com ODS 9 e impacto na Terra (rubrica: 3 pts)

A mesma estratégia de otimização que mantém **satélites operando anos sem reinício físico** habilita **estações meteo em fazendas familiares operarem 24/7 sem manutenção**, em locais sem energia confiável. Isso democratiza acesso à tecnologia espacial: o pequeno agricultor do sertão de Pernambuco recebe a mesma qualidade de dado climático que produtor de soja em MT.

**ODS 9 (Infraestrutura)** materializado: infraestrutura digital resiliente em áreas remotas.

### 4.4 Mobile — App acessível (React Native + Expo)

**Entrega:** App funcional como **dashboard central que conecta as disciplinas** (link Whimsical no PDF).

#### Telas principais

**1. Home — "Sua roça hoje"**
```
┌─────────────────────────────────┐
│  🌳 2F-AGRO       ☀️ 27°C  🔔  │
├─────────────────────────────────┤
│  Bom dia, Seu João! 👋          │
│                                 │
│  ╔═══════════════════════════╗  │
│  ║   ⚠️  ATENÇÃO HOJE        ║  │
│  ║                           ║  │
│  ║  Chance de seca: ALTA     ║  │
│  ║  Próximos 7 dias          ║  │
│  ║                           ║  │
│  ║  💧 Sugestão:             ║  │
│  ║     Irrigar Lavoura 2     ║  │
│  ║                           ║  │
│  ║    [ 🔊 OUVIR ]           ║  │
│  ╚═══════════════════════════╝  │
│                                 │
│  ┌─ MINHAS LAVOURAS ─────────┐ │
│  │ 🌽 Milho - L1     🟢 OK   │ │
│  │ 🍅 Tomate - L2    🟡 SECA │ │
│  │ 🥬 Alface - L3    🟢 OK   │ │
│  └───────────────────────────┘ │
│                                 │
│  [📷 Tirar foto da folha]       │
│  [🗺️ Ver mapa da região]       │
│  [👥 Cooperativa]               │
│                                 │
│  [🏠] [📊] [📷] [💬] [⚙️]      │
└─────────────────────────────────┘
```

**2. Tirar foto da folha (conecta com IoT/CV)** — expo-camera → modelo ONNX local → resultado:
```
"Detectamos: Mancha Bacteriana
 Confiança: 87%
 O que fazer: Remover folhas atingidas
 e aplicar [recomendação]
 [🔊 OUVIR] [👨‍🌾 Falar com agrônomo]"
```

**3. Detalhes da Lavoura** — gráfico de NDVI, histórico, probabilidade, cluster K-Means da propriedade

**4. Mapa da Região (cooperativa)** — Leaflet com pinos coloridos das propriedades

**5. Cooperativa / Comunidade** — feed de alertas regionais, chat com agrônomo

**6. Configurações** — perfil, idioma (PT-BR base), tamanho de fonte, sons

#### Stack
- Expo SDK 51+ (managed workflow)
- React Navigation 6
- React Query (cache offline) + Zustand
- expo-camera + expo-location
- react-native-maps (Leaflet adapter)
- expo-notifications (push)
- expo-av (player de áudio pros TTS)
- AsyncStorage pra dados offline

#### Princípios de acessibilidade (foco do pivot)

- ✅ **Tamanho de fonte:** mínimo 16pt; escalável a 1.5x via config
- ✅ **Contraste:** WCAG AA mínimo (4.5:1) em todo texto
- ✅ **Cores semafóricas:** 🟢🟡🔴 + ícone (não depende só da cor)
- ✅ **Texto curto:** frases de até 12 palavras
- ✅ **Áudio:** botão "Ouvir" em todo alerta (expo-speech pra TTS)
- ✅ **Offline-first:** funciona 100% offline, sincroniza ao pegar 4G
- ✅ **Baixo dado:** imagens otimizadas WebP, JSON gzip, < 5 MB/mês
- ✅ **Compatibilidade:** target Android 7.0+ (cobertura > 90% dos smartphones BR)

### 4.5 C# .NET — Serviço de Alertas Agro (100 pts)

**Entrega:** Projeto .NET 8 Web API + Console de demonstração + README + diagrama + evidências de execução

#### Modelagem POO completa (rubrica de 20pts cumprida)

```csharp
namespace FiapAgro.Domain;

// Abstrata + herança + polimorfismo (POO 20pts)
public abstract class Alerta
{
    public Guid Id { get; init; } = Guid.NewGuid();
    public DateTime CriadoEm { get; init; } = DateTime.UtcNow;
    public string PropriedadeId { get; set; } = string.Empty;
    public double Probabilidade { get; set; }
    public abstract string TipoEvento { get; }
    public abstract NivelSeveridade CalcularSeveridade();
    public abstract string Recomendacao();
}

public class AlertaPraga : Alerta {
    public string EspeciePraga { get; set; } = string.Empty;
    public string CulturaAfetada { get; set; } = string.Empty;
    public override string TipoEvento => "PRAGA";
    public override NivelSeveridade CalcularSeveridade() =>
        Probabilidade > 0.85 ? NivelSeveridade.Critico
        : Probabilidade > 0.6 ? NivelSeveridade.Alto
        : NivelSeveridade.Medio;
    public override string Recomendacao() =>
        $"Inspecionar cultura {CulturaAfetada}. Possível {EspeciePraga}.";
}

public class AlertaSeca : Alerta {
    public double DiasSemChuva { get; set; }
    public override string TipoEvento => "SECA";
    public override NivelSeveridade CalcularSeveridade() =>
        DiasSemChuva > 21 ? NivelSeveridade.Critico : NivelSeveridade.Alto;
    public override string Recomendacao() =>
        $"Irrigação recomendada. {DiasSemChuva:F0} dias sem chuva.";
}

public class AlertaGeada : Alerta { /* ... */ }
public class AlertaEnchente : Alerta { /* ... */ }
public class AlertaErosao : Alerta { /* ... */ }

// Interface + Injeção de Dependência (Abstração 20pts)
public interface IDetector<T> where T : Alerta
{
    Task<T?> DetectarAsync(DadosTelemetria dados, CancellationToken ct);
}

public interface INotificador
{
    Task EnviarAsync(Alerta alerta, IEnumerable<Destinatario> destinos);
}

// Classe abstrata adicional pra detector
public abstract class DetectorBase<T> : IDetector<T> where T : Alerta
{
    protected readonly ILogger _logger;
    protected DetectorBase(ILogger logger) => _logger = logger;
    public abstract Task<T?> DetectarAsync(DadosTelemetria dados, CancellationToken ct);
    protected bool DadosValidos(DadosTelemetria d) =>
        d != null && d.Timestamp > DateTime.UtcNow.AddDays(-1);
}

// Struct + Partial (5pts)
public readonly struct Coordenada
{
    public double Latitude { get; init; }
    public double Longitude { get; init; }
    public override string ToString() => $"{Latitude:F4}, {Longitude:F4}";
}

public partial class Propriedade
{
    public Guid Id { get; init; }
    public string Nome { get; set; } = string.Empty;
    public Coordenada Localizacao { get; set; }
    public double AreaHectares { get; set; }
}

public partial class Propriedade  // métodos calculados
{
    public NivelRisco AvaliarRiscoAtual(IEnumerable<Alerta> alertas)
    {
        var ativos = alertas.Where(a =>
            a.CriadoEm > DateTime.UtcNow.AddHours(-24)).ToList();
        if (ativos.Any(a => a.CalcularSeveridade() == NivelSeveridade.Critico))
            return NivelRisco.Critico;
        // ...
    }
}

public enum NivelSeveridade { Baixo, Medio, Alto, Critico }
public enum NivelRisco { Seguro, Atencao, Alerta, Critico }
```

#### Tratamento de exceções (10pts — "sistemas críticos não podem quebrar")

```csharp
public async Task<IActionResult> ProcessarTelemetriaAsync(DadosTelemetria dados)
{
    try
    {
        var alerta = await _detectorPraga.DetectarAsync(dados, _ct);
        if (alerta != null)
            await _notificador.EnviarAsync(alerta, _destinatarios);
        return Ok(new { processado = true, alertaGerado = alerta != null });
    }
    catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.ServiceUnavailable)
    {
        _logger.LogWarning(ex, "ML Service indisponível, usando fallback");
        return await ProcessarComRegrasFallback(dados);  // não quebra
    }
    catch (ValidationException ex)
    {
        _logger.LogError(ex, "Dados de telemetria inválidos");
        return BadRequest(new { erro = ex.Message });
    }
    catch (Exception ex)
    {
        _logger.LogCritical(ex, "Erro inesperado processando telemetria");
        return StatusCode(500, new { erro = "Erro interno" });
    }
}
```

#### Estrutura de pastas
```
FiapAgro.Api/
├── Controllers/
│   ├── PropriedadesController.cs
│   ├── AlertasController.cs
│   └── DiagnosticoController.cs
├── Domain/        # classes, interfaces, enums
├── Services/      # implementações
├── Infrastructure/  # EF Core, RabbitMQ
├── Tests/
├── README.md
└── docs/
    └── fluxograma.png
```

### 4.6 Cybersecurity — Threat Model + Compliance (10 pts)

#### 1. Threat Modeling

**Ativos críticos:**
- 📍 Dados de geolocalização das propriedades (LGPD — sensível)
- 💰 Dados financeiros (renda, área, produção) — confidenciais
- 🧠 Modelo ML treinado (alvo de envenenamento)
- 🗄️ Banco de alertas (integridade)
- 📡 Estação edge IoT (acesso físico em fazenda)
- 🔌 API Gateway (disponibilidade crítica em época de safra)

**3 vetores de ataque (rubrica pede 3):**

1. **Interceptação de telemetria edge** — atacante intercepta dados entre estação IoT e cloud (man-in-the-middle), revelando localização exata de propriedades e podendo vender pra concorrência ou ladrões de safra
2. **Manipulação de telemetria** — atacante envia dados falsos pra estação (ex: simula chuva) → modelo prevê erradamente → agricultor não irriga e perde safra (envenenamento indireto)
3. **Vazamento massivo de PII via API** — exploração de endpoint não autenticado lista todos os agricultores da plataforma → violação LGPD massiva + risco físico (roubo, sequestro)

#### 2. Arquitetura de Segurança

| Pilar | Controles |
|---|---|
| **Controles de Acesso** | MFA pra cooperativas e admins; JWT com claims (`role:agricultor`, `role:cooperativa`, `role:emater`); RBAC restritivo; privilégio mínimo |
| **Proteção de Dados** | TLS 1.3 em trânsito; AES-256 em repouso (Postgres TDE); bcrypt em senhas; **anonimização de coordenadas em logs** (truncar a 2 casas decimais ≈ 1 km); criptografia em backups |
| **Infraestrutura** | Zero Trust (validar cada request); WAF (Cloudflare/Azure); SIEM (Grafana + Loki + Promtail); secrets em Vault; certificate pinning no app mobile; rate limit 100 req/min por IP |

#### 3. Governança e Compliance

- **ISO 27001:** matriz de risco × probabilidade × impacto; SGSI formal; auditoria semestral; política de senhas; gestão de incidentes
- **LGPD:** consentimento explícito pra coleta de localização e dados produtivos; direito ao esquecimento implementado (endpoint `DELETE /me`); DPO designado; RIPD (Relatório de Impacto à Proteção de Dados) escrito; minimização de coleta

#### 4. Plano de Resposta a Incidentes (3 pts — vale mais)

```
┌──────────────────────────────────────────────────────────┐
│ FASE 1: CONTENÇÃO (0-30 min)                             │
├──────────────────────────────────────────────────────────┤
│ • Isolar serviço comprometido (k8s NetworkPolicy bloqueia)│
│ • Revogar todos os tokens JWT (rotation forçada)         │
│ • Cortar uplink suspeito da estação IoT identificada     │
│ • Ativar modo degradado (read-only) na API               │
│ • Notificar time de IR e DPO                             │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ FASE 2: ERRADICAÇÃO (30 min - 4h)                        │
├──────────────────────────────────────────────────────────┤
│ • Análise forense de logs (Grafana + Loki)               │
│ • Identificar IoC (Indicators of Compromise)             │
│ • Patch da vulnerabilidade                               │
│ • Re-treinar modelo ML se houver envenenamento           │
│ • Rotação de chaves e credentials                        │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ FASE 3: RECUPERAÇÃO (4h - 48h)                           │
├──────────────────────────────────────────────────────────┤
│ • Restore de backup verificado (último limpo)            │
│ • Validação de integridade do banco                      │
│ • Comunicação a stakeholders e ANPD (se vazamento PII)   │
│ • Reativação gradual dos serviços                        │
│ • Post-mortem documentado                                │
│ • Atualização do threat model                            │
└──────────────────────────────────────────────────────────┘
```

### 4.7 SOA — Service-Oriented Architecture

> ⚠️ **Escopo atualizado em 28/05** — o professor de SOA passou a exigir **implementação real** (código), não só documento. SOA virou um projeto próprio em **Java + Spring Boot**, no repo [2f-agro-soa](https://github.com/GS-SPACE-CONNECT/2f-agro-soa), separado do backend C#. Épico: `2f-agro-soa#1`.

**Entrega:** Código-fonte (Java) + Documento PDF com evidências.

**Rubrica (25% cada):** API REST · Web Service SOAP · Integração entre serviços · Documentação da arquitetura.

**Stack:** Java 17, Spring Boot 3, Spring Web (REST), Spring-WS (SOAP contract-first + WSDL), Spring Data JPA + H2.

**Solução (narrativa Space Connect):**
App moderno (**REST**) ↔ sistema legado de governo simulado (**SOAP** — cadastro rural tipo EMATER/MAPA/CAR) + enriquecimento com **dados espaciais externos** (NASA POWER / CPTEC-INPE). Entidade principal: `Propriedade`.

**Componentes:**
1. **API REST (Spring Boot)** — CRUD completo de `Propriedade` (GET/POST/PUT/DELETE, JSON, Bean Validation, `@ControllerAdvice`).
2. **Web Service SOAP (Spring-WS)** — contract-first (XSD → WSDL); operação de consulta (`consultarCadastroRural`) + operação de cadastro/processamento (`registrarCadastroRural`); testado no SoapUI.
3. **Integração** — (a) REST consome **NASA POWER/CPTEC** (clima por lat/long); (b) cliente REST↔SOAP interno (cadastro valida no "governo").
4. **POO obrigatória (Java)** — `abstract Alerta` → `AlertaSeca/Geada/Praga` (herança+polimorfismo); `interface ServicoClimatico` (abstração); entidades encapsuladas.
5. **Persistência** — H2 + Spring Data JPA, CRUD persistente.
6. **Diferenciais (bônus):** Swagger/OpenAPI, Docker Compose, testes JUnit, mensageria.

**Conteúdo do PDF:** integrantes+RM, problema/objetivos, **diagrama de arquitetura SOA**, explicação REST (endpoints+exemplos), explicação SOAP (operações+XML+WSDL), explicação da integração, evidências/prints, tecnologias, conclusão.

> 📄 A visão conceitual anterior (microsserviços .NET, mensageria, observabilidade) continua válida como **arquitetura macro** do 2F-AGRO no ArchiMate/QA — mas a **entrega avaliada de SOA** é o projeto Java acima.

### 4.8 QA — ArchiMate + Backlog + Pitch (100 pts)

#### ArchiMate (40 pts) — 4 camadas

**Camada de Motivação:**
- Stakeholders: Agricultor familiar, Cooperativa, EMATER, MAPA, ONGs
- Objetivos: reduzir perda de safra; democratizar tecnologia; aumentar resiliência climática
- Drivers: ODS 2/1/8/13/15; agricultura familiar = 70% do alimento BR
- Restrições: dataset simulado, equipe de 5, prazo 09/06, acessibilidade obrigatória

**Camada de Negócio:**
- Atores: Agricultor, Gestor de Cooperativa, Extensionista EMATER
- Roles: Produtor, Gestor, Agrônomo, Admin
- Processos: Monitorar Propriedade → Detectar Risco → Confirmar (foto) → Recomendar Ação → Acompanhar Resultado
- Serviços de Negócio: Diagnóstico Agronômico, Alerta Preventivo, Recomendação Inteligente, Visão Coletiva

**Camada de Aplicação:**
- Componentes: App Mobile, Web Dashboard, API Gateway, ML Service, Visão Service, Alertas Service, Notificação, Auth, Ingestão
- Integrações: REST + MQ + Webhooks

**Camada de Tecnologia:**
- React Native + Expo (mobile)
- .NET 8 + EF Core (back C#)
- Python + FastAPI + scikit-learn + Ultralytics (ML/CV)
- PostgreSQL + PostGIS (geo)
- RabbitMQ (mensageria)
- Raspbian + sensores (edge)
- Azure / AWS (cloud)

#### Product Backlog (40 pts) — Épicos

1. **Modelagem de Risco Agronômico (ML)**
2. **Coleta de Dados Multi-camada (Edge + Satélite + Câmera)**
3. **App Mobile Acessível (Field App)**
4. **Dashboard Cooperativa (Web)**
5. **Alertas e Recomendações Inteligentes**
6. **Segurança, LGPD e Compliance**
7. **Identificação de Pragas via IA (CV)**
8. **Visão Coletiva Regional**

**Exemplos de histórias:**

> *Como agricultor familiar com baixa alfabetização, quero receber alertas em formato de áudio na minha língua, para entender o risco mesmo sem saber ler.*
> **Critérios de aceite:** botão "Ouvir" em todo alerta; voz em pt-BR; áudio gerado pelo TTS local (expo-speech); funciona offline.
> **Prioridade:** ALTA. **Story Points:** 5.

> *Como produtor de tomate, quero fotografar uma folha suspeita pra saber se é praga, para agir antes de espalhar.*
> **Critérios de aceite:** abre câmera; processa em < 3s no celular; mostra nome + confiança; sugere ação; salva histórico.
> **Prioridade:** ALTA. **Story Points:** 13.

> *Como gestor de cooperativa, quero ver o mapa das propriedades dos meus associados com cores de risco, para coordenar ação preventiva.*
> **Critérios de aceite:** dashboard web; cada propriedade um pino; cor pelo nível; filtro por cultura; export CSV.
> **Prioridade:** MÉDIA. **Story Points:** 8.

(Backlog completo terá ~20-25 histórias.)

#### Vídeo Pitch (20 pts) — Roteiro de 3 min

| Tempo | Conteúdo |
|---|---|
| **0:00-0:15** | **Hook:** Seu João, agricultor familiar do sertão, perde safra de feijão por geada que ninguém avisou |
| **0:15-0:35** | **Problema:** 77% das propriedades rurais BR = pequenas; 70% do alimento que comemos vem delas; 0% de tecnologia agrícola disponível pra elas |
| **0:35-1:15** | **Solução:** 2F-AGRO — 3 camadas (satélite + estação solar + foto da folha) → IA → recomendação simples no celular do Seu João |
| **1:15-2:00** | **Demo:** abre app, mostra "Atenção: seca chegando", botão áudio, foto da folha detecta praga, mapa da cooperativa |
| **2:00-2:30** | **Tecnologia:** stack completa, ODS 2/1/8/13/15, LGPD, integração EMATER |
| **2:30-2:50** | **Modelo de negócio:** SaaS B2B (cooperativa paga R$ X/mês), B2G (parceria com EMATER/MAPA via Pronaf), impacto social mensurável |
| **2:50-3:00** | **Call to action:** "Pequeno produtor merece tecnologia grande. 2F-AGRO." |

---

## 5. Cronograma (13 dias até 09/06)

| Período | Marco | Responsável |
|---|---|---|
| 27/05 (hoje) | ✅ Spec aprovado | Time |
| 28-29/05 | Setup repos Git + ArchiMate v1 + Backlog v1 + Wireframes mobile | Negócio + Mobile |
| 28-29/05 | Notebook ML — etapas 1-3 (EDA + Reg.Linear) | Python/IA |
| 30/05-01/06 | Notebook ML — etapas 4-6 (LogReg + Tree + KMeans) + relatório draft | Python/IA |
| 30/05-01/06 | C# API esqueleto + domínio + EF Core | C#/.NET |
| 30/05-01/06 | Mobile esqueleto + navegação + telas básicas | Mobile |
| 02-04/06 | Integração ML↔C#↔Mobile via REST (contratos OpenAPI) | Time técnico |
| 02-04/06 | YOLO/CV treinado + script Python + integração mobile | Python/IA + Mobile |
| 02-04/06 | OS doc (Raspbian + tuning) — 4h estimadas | Python/IA |
| 05-06/06 | Cyber threat model + Compliance doc | Negócio + C# |
| 30/05-03/06 | ⚠️ **SOA Java — implementação** (setup + REST CRUD + SOAP/WSDL + integração NASA/CPTEC) — *escopo novo de 28/05, ver § 4.7* | brunão + ruan |
| 04-05/06 | SOA testes (SoapUI + REST) + evidências + PDF SOA | brunão + ruan |
| 05-06/06 | Polimento ArchiMate v2 + Backlog v2 | Negócio |
| 07/06 | Gravação pitch 3min + Revisão final integrada | Negócio + Time |
| 08/06 | **Buffer pra correções** (CRÍTICO — não pular) | Todos |
| 09/06 | **Entrega final** até 23:59 no Teams | Líder do grupo |

---

## 6. Divisão sugerida de tarefas (5 pessoas)

> ⚠️ Substituir Pessoa A/B/C/D/E pelos nomes reais do time

| Pessoa | Perfil dominante | Matéria primária (responsável final) | Matéria secundária (apoio) |
|---|---|---|---|
| **Pessoa A** | Python/IA #1 | **ML** (Notebook completo + relatório) | OS (doc) |
| **Pessoa B** | Python/IA #2 | **IoT / Visão Computacional** (YOLO + script + vídeo) | ML (apoio) |
| **Pessoa C** | C#/Backend | **C# .NET** (API + POO + Console) | SOA (doc) |
| **Pessoa D** | Mobile (RN+Expo) | **Mobile** (App acessível + integrações) | IoT (chamada CV) |
| **Pessoa E** | Negócio/Design/Pitch | **QA** (ArchiMate + Backlog + Pitch) + **Cyber** (doc completo) | SOA (apoio) |

**Princípio:** cada matéria tem **1 responsável final** (defende perante banca), mas o time todo revisa antes da entrega. Pessoa E carrega o trabalho coletivo no ArchiMate.

---

## 7. Riscos e mitigação

| Risco | Probab. | Impacto | Mitigação |
|---|---|---|---|
| Modelo ML com R² baixo | Média | Alto | Plano B: Random Forest; engenharia de features (NDVI × umidade) |
| Integração ML↔C# atrasa | Alta | Alto | Definir contrato OpenAPI dia 30/05; mock no início |
| Modelo de pragas não detecta confiável | Média | Médio | Modelo pré-treinado (PlantVillage); thresholds conservadores; vídeo controlado |
| Atraso no ArchiMate | Média | Alto | Começar 28/05; tutorial Archi 5.x; usar exemplos do livro Hoogendoorn |
| Bug crítico véspera | Média | Alto | Buffer 08/06; daily check; smoke test integrado |
| Integrante doente | Baixa | Médio | Cross-training (pair programming); cada matéria tem secundário |
| Pitch ultrapassar 3 min | Alta | Médio | Roteiro escrito; ensaio cronometrado 2x; cortar excesso |

---

## 8. Próximos passos imediatos (esta semana)

- [ ] Confirmar spec com o grupo todo
- [ ] Substituir Pessoa A/B/C/D/E pelos nomes reais
- [ ] Criar repositório Git compartilhado (GitHub Organization "2F-AGRO")
- [ ] Setup Microsoft Teams / Discord / WhatsApp do time
- [ ] Marcar reunião de planning (28/05)
- [ ] Instalar ferramentas: Archi 5.9 (já está em Downloads), VSCode + extensões, .NET 8 SDK, Python 3.10, Node 18+, Android Studio
- [ ] Pessoa A baixa o CSV no Colab e roda Etapa 1

---

**Documento gerado em:** 27/05/2026
**Versão:** v2.0 — Pivot pra foco agro pequeno produtor (substitui v1 ARGUS-BR)
**Status:** Aguardando review do grupo
