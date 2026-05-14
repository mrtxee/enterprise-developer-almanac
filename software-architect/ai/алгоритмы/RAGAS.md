---
aliases:
  - RAGAS
  - Retrieval-Augmented Generation Assessment
  - Retrieval-Augmented Generation
---
# RAGAS Framework - полное руководство

## Что такое RAGAS?

**RAGAS** (Retrieval-Augmented Generation Assessment) - это специализированный фреймворк для **оценки качества систем RAG** (Retrieval-Augmented Generation). RAGAS предоставляет метрики и инструменты для измерения эффективности RAG-систем, которые объединяют поиск информации и генерацию текста.

## Архитектура RAGAS

### **Основные компоненты RAGAS**
```
┌─────────────────────────────────────────────────────────────┐
│                    RAG Pipeline Evaluation                   │
├─────────────────────────────────────────────────────────────┤
│  Input:                                                     │
│  • Question (вопрос пользователя)                          │
│  • Ground Truth (ожидаемый ответ, если доступен)           │
│  • Retrieved Documents (документы из retriever)            │
│  • Generated Answer (сгенерированный ответ LLM)            │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                    RAGAS Evaluation Framework               │
├──────────────┬──────────────┬──────────────┬───────────────┤
│ Faithfulness │ Answer       │ Context      │ Aspect-       │
│ (Верность)   │ Relevance    │ Relevance    │ Specific      │
│              │ (Релевант-   │ (Релевант-   │ Metrics       │
│              │  ность ответа)│ ность контекста)│              │
└──────────────┴──────────────┴──────────────┴───────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                 Quantitative Metrics                        │
│                 • Scores (0-1 или 0-5)                      │
│                 • Statistical Analysis                      │
│                 • Visualization                             │
└─────────────────────────────────────────────────────────────┘
```

## Ключевые метрики RAGAS

### **1. Faithfulness (Верность/Фактическая точность)**
**Измеряет**: Насколько сгенерированный ответ соответствует предоставленному контексту (без галлюцинаций)

```python
from ragas.metrics import faithfulness
from ragas import evaluate
import datasets

# Пример вычисления faithfulness
dataset = datasets.Dataset.from_dict({
    'question': ['Какая столица Франции?'],
    'answer': ['Столица Франции - Париж.'],
    'contexts': [['Франция - страна в Западной Европе. Столица - Париж.']]
})

score = evaluate(
    dataset,
    metrics=[faithfulness]
)

print(f"Faithfulness score: {score['faithfulness']}")
# Высокий score если ответ соответствует контексту
```

### **2. Answer Relevance (Релевантность ответа)**
**Измеряет**: Насколько ответ соответствует вопросу

```python
from ragas.metrics import answer_relevancy

dataset = datasets.Dataset.from_dict({
    'question': ['Как работают нейронные сети?'],
    'answer': ['Нейронные сети - это вычислительные модели, вдохновленные биологическими нейронными сетями.'],
    'contexts': [['Нейронные сети состоят из слоев нейронов...']]
})

score = evaluate(
    dataset,
    metrics=[answer_relevancy]
)
```

### **3. Context Relevance (Релевантность контекста)**
**Измеряет**: Насколько извлеченные документы релевантны вопросу

```python
from ragas.metrics import context_relevancy

dataset = datasets.Dataset.from_dict({
    'question': ['Каковы преимущества машинного обучения?'],
    'answer': ['МЛ позволяет автоматизировать сложные задачи...'],
    'contexts': [[
        'Машинное обучение используется для прогнозирования.',
        'Машинное обучение требует больших данных.',
        'История машинного обучения началась в 1950-х годах.'
    ]]
})

score = evaluate(
    dataset,
    metrics=[context_relevancy]
)
```

### **4. Context Precision/Recall**
```python
from ragas.metrics import context_precision, context_recall

# Для случаев с ground truth
dataset = datasets.Dataset.from_dict({
    'question': ['Что такое RAG?'],
    'answer': ['RAG - это Retrieval-Augmented Generation...'],
    'contexts': [['RAG объединяет поиск и генерацию...']],
    'ground_truth': [['RAG - архитектура, сочетающая поиск информации и генерацию текста.']]
})

score = evaluate(
    dataset,
    metrics=[context_precision, context_recall]
)
```

## Полный процесс оценки с RAGAS

### **Пример end-to-end оценки**
```python
import pandas as pd
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_relevancy,
    context_precision,
    answer_correctness
)
from datasets import Dataset

# 1. Подготовка данных для оценки
data = {
    'question': [
        'Какие основные компоненты RAG системы?',
        'Как работает attention в transformer?',
        'Что такое fine-tuning в NLP?'
    ],
    'answer': [
        'RAG система состоит из retriever и generator компонентов.',
        'Attention mechanism позволяет модели фокусироваться на разных частях входных данных.',
        'Fine-tuning - это дообучение предварительно обученной модели на специфичных данных.'
    ],
    'contexts': [
        ['RAG (Retrieval-Augmented Generation) объединяет поисковую систему и языковую модель.'],
        ['Transformer использует механизм внимания для обработки последовательностей.'],
        ['Fine-tuning адаптирует общую модель к конкретной задаче или домену.']
    ],
    'ground_truth': [
        ['Основные компоненты: retriever (поиск) и generator (генерация).'],
        ['Attention вычисляет взвешенную сумму значений на основе запроса и ключей.'],
        ['Fine-tuning - процесс дообучения модели на целевом наборе данных.']
    ]
}

# 2. Создание датасета
dataset = Dataset.from_dict(data)

# 3. Определение метрик для оценки
metrics = [
    faithfulness,
    answer_relevancy, 
    context_relevancy,
    context_precision,
    answer_correctness
]

# 4. Запуск оценки
results = evaluate(dataset, metrics=metrics)

# 5. Анализ результатов
results_df = results.to_pandas()
print(results_df)

# Средние значения метрик
print(f"\nAverage Scores:")
print(f"Faithfulness: {results_df['faithfulness'].mean():.3f}")
print(f"Answer Relevancy: {results_df['answer_relevancy'].mean():.3f}")
print(f"Context Relevancy: {results_df['context_relevancy'].mean():.3f}")
```

### **Интеграция с RAG pipeline**
```python
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings
from langchain.llms import OpenAI
from ragas.llms import LangchainLLM
from ragas.metrics import answer_correctness

class RAGPipelineEvaluator:
    def __init__(self, vector_store_path, llm_model="gpt-3.5-turbo"):
        self.embeddings = OpenAIEmbeddings()
        self.vector_store = FAISS.load_local(vector_store_path, self.embeddings)
        self.llm = OpenAI(model_name=llm_model, temperature=0)
        self.evaluator_llm = LangchainLLM(self.llm)
    
    def evaluate_response(self, question, answer, k_documents=3):
        # Получение релевантных документов
        retrieved_docs = self.vector_store.similarity_search(question, k=k_documents)
        contexts = [doc.page_content for doc in retrieved_docs]
        
        # Подготовка данных для оценки
        evaluation_data = {
            'question': [question],
            'answer': [answer],
            'contexts': [contexts]
        }
        
        dataset = Dataset.from_dict(evaluation_data)
        
        # Вычисление метрик
        results = evaluate(
            dataset,
            metrics=[faithfulness, answer_relevancy, context_relevancy],
            llm=self.evaluator_llm
        )
        
        return results
    
    def benchmark_pipeline(self, test_questions, ground_truths):
        """Бенчмаркинг всей RAG системы"""
        scores = []
        
        for question, truth in zip(test_questions, ground_truths):
            # Генерация ответа через RAG систему
            retrieved = self.vector_store.similarity_search(question, k=3)
            context = " ".join([doc.page_content for doc in retrieved])
            
            prompt = f"Контекст: {context}\n\nВопрос: {question}\n\nОтвет:"
            answer = self.llm(prompt)
            
            # Оценка
            eval_data = Dataset.from_dict({
                'question': [question],
                'answer': [answer],
                'contexts': [[retrieved]],
                'ground_truth': [[truth]]
            })
            
            result = evaluate(
                eval_data,
                metrics=[answer_correctness, faithfulness],
                llm=self.evaluator_llm
            )
            
            scores.append({
                'question': question,
                'answer': answer,
                'faithfulness': result['faithfulness'],
                'correctness': result['answer_correctness']
            })
        
        return pd.DataFrame(scores)
```

## Кастомные метрики в RAGAS

### **Создание собственных метрик**
```python
from ragas.metrics.base import Metric
from ragas.llms import llm_factory
from ragas.metrics.critique import CritiqueMetric

class CustomHallucinationMetric(Metric):
    """Кастомная метрика для обнаружения галлюцинаций"""
    
    def __init__(self):
        super().__init__()
        self.name = "hallucination_score"
        self.llm = llm_factory("gpt-4")
    
    def score(self, row):
        question = row["question"]
        answer = row["answer"]
        contexts = row["contexts"]
        
        prompt = f"""
        Оцените, содержит ли ответ галлюцинации (информацию, отсутствующую в контексте).
        
        Вопрос: {question}
        Ответ: {answer}
        Контекст: {' '.join(contexts)}
        
        Оцените от 1 до 5, где:
        1 - Ответ содержит много галлюцинаций
        3 - Ответ содержит некоторые галлюцинации  
        5 - Ответ полностью соответствует контексту
        
        Верните только число.
        """
        
        response = self.llm.generate_text(prompt)
        try:
            score = float(response.strip())
            normalized = (score - 1) / 4  # Нормализация к 0-1
            return normalized
        except:
            return 0.0

# Использование кастомной метрики
custom_metric = CustomHallucinationMetric()
results = evaluate(dataset, metrics=[faithfulness, custom_metric])
```

### **Composite Metrics (Составные метрики)**
```python
from ragas.metrics import MetricWithLLM

class RAGQualityScore(MetricWithLLM):
    """Составная метрика качества RAG"""
    
    def __init__(self, weights=None):
        super().__init__()
        self.name = "rag_quality_score"
        self.weights = weights or {
            'faithfulness': 0.4,
            'answer_relevancy': 0.3,
            'context_relevancy': 0.3
        }
    
    def score(self, row, other_metrics):
        """Вычисление составного score на основе других метрик"""
        total_score = 0
        
        for metric_name, weight in self.weights.items():
            if metric_name in other_metrics:
                metric_score = other_metrics[metric_name]
                total_score += metric_score * weight
        
        return total_score
```

## Best Practices использования RAGAS

### **Настройка оценки для разных сценариев**
```python
class RAGASEvaluationConfig:
    """Конфигурация оценки в зависимости от use case"""
    
    @staticmethod
    def get_config(evaluation_type="standard"):
        configs = {
            "standard": {
                "metrics": ["faithfulness", "answer_relevancy", "context_relevancy"],
                "llm": "gpt-3.5-turbo",
                "thresholds": {
                    "faithfulness": 0.8,
                    "answer_relevancy": 0.7,
                    "context_relevancy": 0.6
                }
            },
            "strict": {
                "metrics": ["faithfulness", "answer_correctness", "context_precision", "context_recall"],
                "llm": "gpt-4",
                "thresholds": {
                    "faithfulness": 0.9,
                    "answer_correctness": 0.85,
                    "context_precision": 0.7,
                    "context_recall": 0.7
                }
            },
            "lightweight": {
                "metrics": ["answer_relevancy", "context_relevancy"],
                "llm": "gpt-3.5-turbo",
                "thresholds": {
                    "answer_relevancy": 0.6,
                    "context_relevancy": 0.5
                }
            }
        }
        return configs.get(evaluation_type, configs["standard"])
    
    def create_evaluator(self, config_name="standard"):
        config = self.get_config(config_name)
        
        # Настройка LLM для оценки
        llm = LangchainLLM(OpenAI(
            model_name=config["llm"],
            temperature=0
        ))
        
        # Выбор метрик
        metrics = []
        for metric_name in config["metrics"]:
            metric_class = self.get_metric_class(metric_name)
            metrics.append(metric_class(llm=llm))
        
        return {
            "evaluator": lambda dataset: evaluate(dataset, metrics=metrics),
            "thresholds": config["thresholds"]
        }
```

### **Автоматизированная оценка пайплайна**
```python
import json
from datetime import datetime

class AutomatedRAGEvaluator:
    def __init__(self, rag_pipeline, evaluation_config):
        self.pipeline = rag_pipeline
        self.evaluator_config = evaluation_config
        self.results_history = []
    
    def run_evaluation_batch(self, test_dataset, batch_size=10):
        """Запуск оценки на батче данных"""
        results = []
        
        for i in range(0, len(test_dataset), batch_size):
            batch = test_dataset[i:i+batch_size]
            batch_results = self._evaluate_batch(batch)
            results.extend(batch_results)
            
            # Логирование прогресса
            self._log_batch_results(batch_results, i)
        
        # Агрегация результатов
        aggregated = self._aggregate_results(results)
        self.results_history.append({
            "timestamp": datetime.now().isoformat(),
            "results": aggregated
        })
        
        return aggregated
    
    def _evaluate_batch(self, batch):
        batch_results = []
        
        for item in batch:
            # Получение ответа от RAG пайплайна
            question = item["question"]
            answer, contexts = self.pipeline.generate(question)
            
            # Подготовка данных для RAGAS
            eval_data = {
                "question": [question],
                "answer": [answer],
                "contexts": [contexts],
                "ground_truth": [item.get("ground_truth", "")]
            }
            
            dataset = Dataset.from_dict(eval_data)
            
            # Запуск оценки
            scores = evaluate(
                dataset,
                metrics=self.evaluator_config["metrics"]
            )
            
            batch_results.append({
                "question": question,
                "answer": answer,
                "scores": scores,
                "contexts": contexts
            })
        
        return batch_results
    
    def generate_report(self):
        """Генерация детального отчета"""
        if not self.results_history:
            return "No evaluation results available"
        
        latest = self.results_history[-1]["results"]
        
        report = {
            "summary": {
                "total_evaluations": latest["count"],
                "average_scores": latest["averages"],
                "passing_rate": self._calculate_passing_rate(latest),
                "weakest_metric": min(latest["averages"], key=latest["averages"].get)
            },
            "detailed_scores": latest["detailed"],
            "recommendations": self._generate_recommendations(latest)
        }
        
        return json.dumps(report, indent=2, ensure_ascii=False)
```

## Визуализация результатов

### **Dashboard с результатами оценки**
```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

class RAGASVisualizer:
    @staticmethod
    def plot_metrics_comparison(results_df):
        """Визуализация сравнения метрик"""
        fig, axes = plt.subplots(2, 2, figsize=(12, 10))
        
        # 1. Распределение scores
        metrics = ['faithfulness', 'answer_relevancy', 'context_relevancy', 'answer_correctness']
        
        for idx, metric in enumerate(metrics):
            ax = axes[idx // 2, idx % 2]
            if metric in results_df.columns:
                sns.histplot(results_df[metric], bins=20, ax=ax, kde=True)
                ax.set_title(f'Distribution of {metric}')
                ax.set_xlabel('Score')
                ax.set_ylabel('Frequency')
                ax.axvline(results_df[metric].mean(), color='red', linestyle='--', 
                          label=f'Mean: {results_df[metric].mean():.2f}')
                ax.legend()
        
        plt.tight_layout()
        return fig
    
    @staticmethod
    def create_correlation_matrix(results_df):
        """Матрица корреляции между метриками"""
        numeric_cols = results_df.select_dtypes(include=['float64', 'int64']).columns
        correlation_matrix = results_df[numeric_cols].corr()
        
        fig, ax = plt.subplots(figsize=(10, 8))
        sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', 
                   center=0, ax=ax, fmt='.2f')
        ax.set_title('Correlation between RAGAS Metrics')
        return fig
    
    @staticmethod
    def plot_trend_over_time(history_data):
        """Визуализация трендов во времени"""
        timestamps = [h["timestamp"] for h in history_data]
        metrics_data = {
            "faithfulness": [h["results"]["averages"]["faithfulness"] for h in history_data],
            "answer_relevancy": [h["results"]["averages"]["answer_relevancy"] for h in history_data]
        }
        
        df_trend = pd.DataFrame({
            "timestamp": pd.to_datetime(timestamps),
            **metrics_data
        })
        
        fig, ax = plt.subplots(figsize=(12, 6))
        for metric in metrics_data.keys():
            ax.plot(df_trend["timestamp"], df_trend[metric], marker='o', label=metric)
        
        ax.set_xlabel('Evaluation Date')
        ax.set_ylabel('Average Score')
        ax.set_title('RAG Performance Trends Over Time')
        ax.legend()
        ax.grid(True, alpha=0.3)
        
        return fig
```

## Интеграция с MLflow для отслеживания экспериментов

```python
import mlflow
from mlflow.models import Model

class RAGASMLflowTracker:
    def __init__(self, experiment_name="RAG-Evaluation"):
        mlflow.set_experiment(experiment_name)
    
    def log_evaluation_run(self, rag_config, evaluation_results, artifacts=None):
        """Логирование результатов оценки в MLflow"""
        with mlflow.start_run():
            # Логирование параметров RAG системы
            mlflow.log_params({
                "retriever_type": rag_config.get("retriever", "unknown"),
                "embedding_model": rag_config.get("embedding", "unknown"),
                "llm_model": rag_config.get("llm", "unknown"),
                "chunk_size": rag_config.get("chunk_size", 0),
                "k_retrieved": rag_config.get("k_retrieved", 0)
            })
            
            # Логирование метрик
            if "averages" in evaluation_results:
                mlflow.log_metrics(evaluation_results["averages"])
            
            # Логирование артефактов
            if artifacts:
                for name, artifact in artifacts.items():
                    if isinstance(artifact, plt.Figure):
                        artifact.savefig(f"{name}.png")
                        mlflow.log_artifact(f"{name}.png")
                    elif isinstance(artifact, pd.DataFrame):
                        artifact.to_csv(f"{name}.csv")
                        mlflow.log_artifact(f"{name}.csv")
            
            # Сохранение модели/пайплайна
            if "pipeline" in rag_config:
                mlflow.pyfunc.log_model(
                    "rag_pipeline",
                    python_model=rag_config["pipeline"],
                    registered_model_name="rag_system"
                )
```

## Пример полного workflow

```python
def complete_rag_evaluation_workflow():
    """
    Полный workflow оценки RAG системы с RAGAS
    """
    # 1. Инициализация компонентов
    rag_pipeline = YourRAGPipeline()
    evaluator = AutomatedRAGEvaluator(
        rag_pipeline,
        evaluation_config=RAGASEvaluationConfig().create_evaluator("standard")
    )
    
    # 2. Загрузка тестовых данных
    test_data = load_test_dataset("path/to/test_data.json")
    
    # 3. Запуск оценки
    results = evaluator.run_evaluation_batch(test_data, batch_size=5)
    
    # 4. Анализ результатов
    visualizer = RAGASVisualizer()
    
    # Создание визуализаций
    results_df = pd.DataFrame(results["detailed"])
    distribution_fig = visualizer.plot_metrics_comparison(results_df)
    correlation_fig = visualizer.create_correlation_matrix(results_df)
    
    # 5. Генерация отчета
    report = evaluator.generate_report()
    
    # 6. Логирование в MLflow
    tracker = RAGASMLflowTracker()
    tracker.log_evaluation_run(
        rag_config=rag_pipeline.get_config(),
        evaluation_results=results,
        artifacts={
            "metrics_distribution": distribution_fig,
            "correlation_matrix": correlation_fig,
            "detailed_results": results_df
        }
    )
    
    # 7. Рекомендации по улучшению
    recommendations = generate_improvement_recommendations(results)
    
    return {
        "report": report,
        "recommendations": recommendations,
        "overall_score": results["averages"]["rag_quality_score"] if "rag_quality_score" in results["averages"] else None
    }
```

RAGAS предоставляет мощный фреймворк для **систематической оценки качества RAG-систем**. Ключевые преимущества:

1. **Комплексные метрики** - покрывают все аспекты качества RAG
2. **Гибкость** - поддержка кастомных метрик и конфигураций
3. **Интеграция** - работает с популярными ML инструментами
4. **Интерпретируемость** - понятные отчеты и визуализации

Это делает RAGAS незаменимым инструментом для разработки, мониторинга и улучшения RAG-приложений в production.