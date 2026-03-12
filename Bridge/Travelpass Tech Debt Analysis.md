
# Updating Bridge CLI functionality

## Setting up Enhanced Metric Parser system
base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Dict, Any, Generator

@dataclass
class MetricResult:
    name: str
    value: float
    weight: float
    category: str
    description: str

class MetricParser(ABC):
    """Base class for all metric parsers"""
    
    @abstractmethod
    def calculate(self, repo_path: str, repo: Any) -> MetricResult:
        """Calculate metric for given repository"""
        pass
    
    @abstractmethod
    def get_weight(self) -> float:
        """Return metric's weight in overall score"""
        pass
    
    @abstractmethod
    def get_threshold(self) -> Dict[str, float]:
        """Return metric's thresholds for recommendations"""
        pass

## Implementing Core Metrics
complexity.py
class ComplexityMetric(MetricParser):
    """Analyzes code complexity using radon"""
    
    def calculate(self, repo_path: str, repo: Any) -> MetricResult:
        complexity_score = self._calculate_complexity(repo_path)
        return MetricResult(
            name="Code Complexity",
            value=complexity_score,
            weight=0.3,
            category="maintainability",
            description="Average cyclomatic complexity across all files"
        )
    
    def get_threshold(self) -> Dict[str, float]:
        return {
            "warning": 7.0,
            "critical": 10.0
        }
## Historical Analysis Implementation
historical.py
class HistoricalAnalyzer:
    """Analyzes repository metrics over time"""
    
    def __init__(self, repo_path: str, metric_parsers: List[MetricParser]):
        self.repo_path = repo_path
        self.metric_parsers = metric_parsers
        
    def analyze_commits(self, num_commits: int = 10) -> Dict[str, List[float]]:
        """Analyze metrics for last N significant commits"""
        trends = defaultdict(list)
        
        for commit in self._get_significant_commits(num_commits):
            self._checkout_commit(commit)
            for parser in self.metric_parsers:
                result = parser.calculate(self.repo_path, None)
                trends[result.name].append(result.value)
                
        return dict(trends)
    
    def calculate_trends(self) -> Dict[str, str]:
        """Calculate trend direction for each metric"""
        trends = {}
        for metric, values in self.historical_data.items():
            trends[metric] = self._calculate_trend(values)
        return trends

## Enhanced Visualization System
visualizer.py
class EnhancedVisualizer:
    """Enhanced visualization system with trends and recommendations"""
    
    def create_detailed_report(self, results: AnalysisResult) -> None:
        """Generate comprehensive analysis report"""
        self._display_header(results)
        self._display_score_breakdown(results.metrics)
        self._display_historical_trends(results.trends)
        self._display_recommendations(results.recommendations)
        
    def _display_historical_trends(self, trends: Dict[str, List[float]]) -> None:
        """Display historical trends using rich tables and ASCII charts"""
        table = Table(title="Historical Trends")
        table.add_column("Metric")
        table.add_column("Trend")
        table.add_column("Change")
        
        for metric, values in trends.items():
            trend = self._calculate_trend_indicator(values)
            change = self._calculate_change_percentage(values)
            table.add_row(
                metric,
                trend,
                f"{change:+.1f}%"
            )

## Recommendation Engine
recommendations.py
class RecommendationEngine:
    """Generates actionable recommendations based on metrics"""
    
    def __init__(self, thresholds: Dict[str, Dict[str, float]]):
        self.thresholds = thresholds
        
    def generate_recommendations(
        self,
        metrics: Dict[str, MetricResult],
        trends: Dict[str, str]
    ) -> List[Recommendation]:
        """Generate prioritized recommendations"""
        recommendations = []
        
        for metric_name, result in metrics.items():
            if recommendation := self._check_metric(result, trends.get(metric_name)):
                recommendations.append(recommendation)
                
        return sorted(recommendations, key=lambda x: x.priority, reverse=True)
    
    def _check_metric(
        self,
        metric: MetricResult,
        trend: Optional[str]
    ) -> Optional[Recommendation]:
        """Check metric against thresholds and generate recommendation"""
        thresholds = self.thresholds.get(metric.name, {})
        
        if metric.value >= thresholds.get("critical", float("inf")):
            return Recommendation(
                title=f"Critical: High {metric.name}",
                description=self._get_recommendation_text(metric, "critical"),
                priority="HIGH",
                effort_estimate="LARGE"
            )
## Integration and Main Analysis Flow
analyzer.py

class EnhancedRepoAnalyzer:
    """Main analyzer combining all components"""
    
    def analyze_repo(self, repo_url: str) -> AnalysisResult:
        """Perform comprehensive repository analysis"""
        
        # Initialize components
        metrics = self._initialize_metric_parsers()
        historical = HistoricalAnalyzer(self.repo_path, metrics)
        recommender = RecommendationEngine(self._get_thresholds(metrics))
        
        # Collect current metrics
        current_metrics = self._collect_metrics(metrics)
        
        # Analyze historical trends
        trends = historical.analyze_commits()
        
        # Generate recommendations
        recommendations = recommender.generate_recommendations(
            current_metrics,
            trends
        )
        
        # Calculate final score
        score = self._calculate_score(current_metrics)
        
        return AnalysisResult(
            score=score,
            metrics=current_metrics,
            trends=trends,
            recommendations=recommendations
        )

## Phases
- Implementation Order:

- First Phase (Core Enhancement):

- Implement base MetricParser system

- Port existing metrics to new system

- Add new metrics from git-code-debt

- Update scoring algorithm

- Second Phase (Historical Analysis):

- Implement HistoricalAnalyzer

- Add commit analysis functionality

- Implement trend calculation

- Add historical data storage

- Third Phase (Visualization):

- Enhance current visualization system

- Add trend visualization

- Implement detailed metric breakdown

- Add chart generation
- - Fourth Phase (Recommendations):

- Implement RecommendationEngine

- Define thresholds and rules

- Add priority calculation

- Integrate with visualization