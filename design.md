# SynapsePath - Design Document

## System Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────────┐
│                         IDE Layer                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Code Editor  │  │ File System  │  │ Debug Panel  │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
└─────────┼──────────────────┼──────────────────┼─────────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
┌────────────────────────────┼─────────────────────────────────┐
│                    SynapsePath Core                          │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────────┐ │
│  │           Event Collection & Processing                 │ │
│  │  • File opens/edits  • Cursor movements  • Errors      │ │
│  └─────────────────────────┬──────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────────┐ │
│  │              Mental Model Engine                        │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐  │ │
│  │  │  Knowledge   │  │  Proficiency │  │  Pattern    │  │ │
│  │  │    Graph     │  │    Tracker   │  │  Detector   │  │ │
│  │  └──────────────┘  └──────────────┘  └─────────────┘  │ │
│  └─────────────────────────┬──────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────────┐ │
│  │         Cognitive Blind Spot Analyzer                   │ │
│  │  • Struggle detection  • Complexity analysis            │ │
│  └─────────────────────────┬──────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────────┐ │
│  │         Micro-Simulation Generator                      │ │
│  │  • Template engine  • Context extraction  • AI model   │ │
│  └─────────────────────────┬──────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────────┐ │
│  │              Learning Interface                         │ │
│  │  • Notification system  • Interactive panel             │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Event Collection & Processing

**Purpose**: Capture all user interactions with the codebase

**Data Collected**:
- File operations (open, edit, save, close)
- Cursor position and dwell time
- Code selections and searches
- Error occurrences and resolution time
- Debug session patterns
- Git operations (commits, diffs viewed)

**Implementation**:
```typescript
interface UserEvent {
  timestamp: number;
  type: EventType;
  context: CodeContext;
  metadata: Record<string, any>;
}

class EventCollector {
  private eventStream: Observable<UserEvent>;
  private buffer: CircularBuffer<UserEvent>;
  
  captureFileEdit(file: string, changes: TextChange[]): void;
  captureCursorDwell(position: Position, duration: number): void;
  captureError(error: Diagnostic, resolutionTime?: number): void;
}
```

### 2. Mental Model Engine

**Purpose**: Build and maintain a dynamic representation of user knowledge

**Knowledge Graph Structure**:
```typescript
interface KnowledgeNode {
  id: string;
  type: 'file' | 'function' | 'class' | 'pattern' | 'library';
  name: string;
  proficiencyScore: number; // 0-100
  lastInteraction: Date;
  interactionCount: number;
  successRate: number;
  dependencies: string[]; // Related node IDs
}

class MentalModelEngine {
  private graph: KnowledgeGraph;
  private proficiencyCalculator: ProficiencyCalculator;
  
  updateProficiency(nodeId: string, event: UserEvent): void;
  getProficiencyMap(): Map<string, number>;
  identifyWeakAreas(): KnowledgeNode[];
  predictNextChallenge(): KnowledgeNode | null;
}
```

**Proficiency Scoring Algorithm**:
- Base score from interaction frequency
- Weighted by success rate (error-free edits)
- Time decay for stale knowledge
- Boost for teaching moments (helping others, writing docs)

### 3. Cognitive Blind Spot Analyzer

**Purpose**: Detect when user is struggling or encountering unfamiliar concepts

**Detection Signals**:
```typescript
interface StruggleSignal {
  type: 'repeated_edits' | 'long_pause' | 'error_cycle' | 
        'frequent_search' | 'context_switching';
  confidence: number;
  affectedNodes: string[];
  suggestedIntervention: InterventionType;
}

class BlindSpotAnalyzer {
  private signalDetectors: SignalDetector[];
  private thresholds: DetectionThresholds;
  
  analyzeRecentActivity(events: UserEvent[]): StruggleSignal[];
  detectUnfamiliarPattern(code: string): Pattern | null;
  calculateInterventionUrgency(signals: StruggleSignal[]): number;
}
```

**Detection Heuristics**:
- Repeated edits in same location (>3 within 5 minutes)
- Cursor dwell time >30s without typing
- Error-fix-error cycle (>2 iterations)
- Rapid file switching (>5 files in 2 minutes)
- Search queries matching unfamiliar APIs

### 4. Micro-Simulation Generator

**Purpose**: Create contextual, interactive learning experiences

**Simulation Types**:
```typescript
type SimulationType = 
  | 'interactive_example'    // Runnable code snippet
  | 'visual_diagram'         // Architecture visualization
  | 'step_by_step_guide'     // Guided walkthrough
  | 'comparison_matrix'      // Pattern alternatives
  | 'debugging_scenario';    // Common pitfalls

interface MicroSimulation {
  id: string;
  type: SimulationType;
  concept: string;
  duration: number; // Target: 30 seconds
  content: SimulationContent;
  interactiveElements: InteractiveElement[];
  successCriteria: SuccessCriteria;
}

class SimulationGenerator {
  private templateLibrary: SimulationTemplate[];
  private codebaseContext: CodebaseAnalyzer;
  private aiModel: LLMInterface;
  
  generateSimulation(
    blindSpot: KnowledgeNode,
    userContext: CodeContext
  ): MicroSimulation;
  
  personalizeContent(
    template: SimulationTemplate,
    userLevel: number
  ): SimulationContent;
}
```

**Generation Pipeline**:
1. Identify core concept from blind spot
2. Extract relevant code examples from codebase
3. Select appropriate simulation template
4. Personalize difficulty based on user proficiency
5. Generate interactive elements
6. Validate 30-second completion time

### 5. Learning Interface

**Purpose**: Present learning opportunities without disrupting flow

**UI Components**:
```typescript
interface LearningNotification {
  priority: 'low' | 'medium' | 'high';
  message: string;
  simulation: MicroSimulation;
  dismissible: boolean;
  timing: 'immediate' | 'next_pause' | 'end_of_session';
}

class LearningInterface {
  private notificationQueue: PriorityQueue<LearningNotification>;
  private panel: InteractivePanel;
  
  showNotification(notification: LearningNotification): void;
  openSimulation(simulation: MicroSimulation): void;
  trackCompletion(simulationId: string, success: boolean): void;
}
```

**Notification Strategy**:
- High priority: Immediate inline suggestion
- Medium priority: Status bar notification
- Low priority: Queue for next natural pause

## Data Models

### User Profile
```typescript
interface UserProfile {
  id: string;
  skillLevel: 'junior' | 'mid' | 'senior';
  knownLanguages: string[];
  knownFrameworks: string[];
  learningPreferences: {
    interventionFrequency: 'minimal' | 'balanced' | 'aggressive';
    preferredSimulationTypes: SimulationType[];
    pauseDetectionThreshold: number;
  };
  statistics: {
    totalSimulationsCompleted: number;
    averageCompletionTime: number;
    conceptsMastered: string[];
    activeBlindSpots: string[];
  };
}
```

### Codebase Model
```typescript
interface CodebaseModel {
  projectId: string;
  languages: string[];
  frameworks: DetectedFramework[];
  architecturalPatterns: Pattern[];
  complexityMap: Map<string, number>;
  dependencyGraph: DependencyGraph;
}
```

## Algorithms

### Proficiency Score Calculation
```
proficiency(node, t) = 
  α * frequency(node, t) * 
  β * successRate(node, t) * 
  γ * recency(node, t) * 
  δ * complexity(node)

where:
  α = frequency weight (0.3)
  β = success weight (0.4)
  γ = recency weight (0.2)
  δ = complexity adjustment (0.1)
  
  recency(node, t) = e^(-λ * daysSinceLastInteraction)
  λ = decay constant (0.05)
```

### Struggle Detection Score
```
struggleScore = Σ(weight_i * signal_i)

Signals:
  - editFrequency: 0.25
  - errorCycleDepth: 0.30
  - dwellTime: 0.20
  - searchFrequency: 0.15
  - contextSwitching: 0.10

Threshold for intervention: struggleScore > 0.65
```

## Technology Stack

### Core Technologies
- **Language**: TypeScript
- **IDE Integration**: VS Code Extension API / JetBrains Plugin SDK
- **Graph Database**: Neo4j (knowledge graph)
- **Time-Series DB**: InfluxDB (event tracking)
- **AI/ML**: OpenAI API / Local LLM (simulation generation)
- **State Management**: Redux Toolkit
- **UI Framework**: React (for panels)

### Storage
- Local SQLite for user profiles and preferences
- IndexedDB for event buffering
- File system for simulation cache

## Security & Privacy

### Data Protection
- All mental model data encrypted at rest
- No code content sent to external services without explicit consent
- Anonymization pipeline for analytics
- User-controlled data retention policies

### Opt-Out Mechanisms
- Global disable switch
- Per-project tracking toggle
- Sensitive file pattern exclusions
- Simulation generation without external AI

## Performance Considerations

### Optimization Strategies
- Event batching (100ms windows)
- Lazy loading of knowledge graph
- Simulation template caching
- Background processing for analysis
- Incremental proficiency updates

### Resource Limits
- Max event buffer: 10,000 events
- Knowledge graph pruning: nodes with proficiency < 10 and no interaction in 30 days
- Simulation cache: 50 most recent

## Future Enhancements

### Phase 2
- Team knowledge sharing
- Collaborative learning sessions
- Integration with code review tools
- Custom simulation authoring

### Phase 3
- Predictive learning paths
- Gamification elements
- Voice-guided simulations
- AR/VR code visualization
