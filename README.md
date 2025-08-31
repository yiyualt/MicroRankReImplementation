# MicroRank Test Case

This test case demonstrates the complete MicroRank workflow using synthetic microservice trace data, allowing you to understand how the system works without needing a real Elasticsearch deployment.

## 🎯 What This Demo Shows

The test case simulates a **microservice e-commerce system** with the following services:
- `frontend` - Web frontend service
- `productcatalogservice` - Product catalog management  
- `currencyservice` - Currency conversion
- `paymentservice` - Payment processing
- `shippingservice` - Shipping calculations
- `emailservice` - Email notifications
- `checkoutservice` - Order checkout
- `recommendationservice` - Product recommendations
- `adservice` - Advertisement service
- `cartservice` - Shopping cart management

## 🔧 Files Overview

| File | Purpose |
|------|---------|
| `mock_data_generator.py` | Generates synthetic distributed traces with realistic latencies |
| `mock_preprocess_data.py` | Mock data preprocessing (replaces Elasticsearch queries) |
| `mock_anormaly_detector.py` | Anomaly detection with configurable thresholds |
| `mock_pagerank.py` | PageRank calculation for service dependency graphs |
| `mock_online_rca.py` | Main RCA engine with spectrum analysis |
| `run_microrank_demo.py` | Demo runner with different modes |

## 🚀 Running the Demo

### Prerequisites

```bash
pip install numpy
```

### Demo Modes

#### 1. Full Continuous Monitoring Demo (Default)
Shows the complete MicroRank workflow with continuous monitoring:

```bash
cd testcase
python run_microrank_demo.py
# or
python run_microrank_demo.py full
```

This demonstrates:
- ✅ Historical data collection and SLO establishment
- 🔍 Continuous system monitoring (every 60 seconds)
- 🚨 Automatic anomaly detection when threshold exceeded
- 📊 Trace partitioning (normal vs anomalous)
- 🧮 PageRank calculation for both trace sets
- 🎯 Spectrum analysis and root cause ranking

#### 2. Quick Analysis Demo
Single analysis cycle focused on the core algorithm:

```bash
python run_microrank_demo.py quick
```

This shows:
- Fast baseline establishment
- Single anomaly detection cycle
- Complete RCA pipeline in one pass

#### 3. Component Testing
Test individual components:

```bash
# Test data generation
python mock_data_generator.py

# Test anomaly detection
python mock_anormaly_detector.py

# Test PageRank calculation
python mock_pagerank.py
```

## 📊 Understanding the Output

### 1. SLO Establishment Phase
```
📊 STEP 1: Data Preprocessing & SLO Establishment
Collected 847 spans
✅ Identified 19 unique operations

Sample SLO profiles (mean latency ± std deviation):
  frontend_Recv: 2.12ms ± 0.85ms
  productcatalogservice_ListProducts: 3.45ms ± 1.23ms
```

### 2. Anomaly Detection
```
🔍 STEP 2: Continuous Monitoring & RCA
[MOCK] anormaly_trace: 12
[MOCK] total_trace: 65
[MOCK] anormaly_rate: 0.185
🚨 ANOMALY DETECTED! Starting root cause analysis...
```

### 3. Root Cause Analysis Results
```
🎯 ROOT CAUSE ANALYSIS RESULTS:
Top 5 suspicious services (ranked by Dstar2 score):
  1. paymentservice-abc123_Charge: 0.845623
  2. shippingservice-def456_ShipOrder: 0.623891
  3. checkoutservice-ghi789_PlaceOrder: 0.445234
  4. emailservice-jkl012_SendOrderConfirmation: 0.234567
  5. frontend-mno345_Recv: 0.123456
```

## 🧮 Algorithm Details

### Data Generation
- **Realistic Latencies**: Each service has normal latency ranges based on typical microservice patterns
- **Anomaly Injection**: Some traces get 3-8x increased latency in critical services
- **Dependency Simulation**: Services call each other following realistic patterns

### Anomaly Detection  
- **SLO Calculation**: Mean + 1.5×standard deviation threshold per operation
- **System Threshold**: >8 anomalous traces triggers system-level anomaly
- **Trace Classification**: Individual traces marked anomalous if total duration > expected

### PageRank Analysis
- **Bipartite Graph**: Services ↔ Traces relationship
- **Weight Calculation**: Based on trace execution patterns
- **Separate Analysis**: Normal traces vs anomalous traces computed independently

### Spectrum Analysis
- **Dstar2 Formula**: `score = ef² / (ep + nf)`
  - `ef`: Executions in Failed traces (anomalous)
  - `ep`: Executions in Passed traces (normal)  
  - `nf`: Non-executions in Failed traces
- **Ranking**: Higher scores indicate more suspicious services

## 🎨 Customization

### Adjust Anomaly Rate
```python
# In mock_data_generator.py, line 163
spans = generator.generate_span_list(start_time, end_time, 
                                   num_traces=100, 
                                   anomaly_rate=0.15)  # 15% anomalous
```

### Change Detection Threshold
```python
# In mock_anormaly_detector.py, line 70
if anormaly_trace > 8:  # Threshold for system anomaly
```

### Modify Spectrum Method
```python
# In mock_online_rca.py, line 242
spectrum_method="dstar2"  # Options: dstar2, ochiai, jaccard, tarantula
```

### Add New Services
```python
# In mock_data_generator.py, lines 6-18
self.services = [
    "frontend", "your_new_service", ...
]

self.operations = {
    "your_new_service": ["Operation1", "Operation2"], ...
}
```

## 🔍 Workflow Deep Dive

1. **Historical Analysis** (60 seconds of data)
   - Collect distributed traces
   - Extract service operations  
   - Calculate normal latency profiles (SLOs)

2. **Real-time Monitoring** (every 60 seconds)
   - Fetch recent traces (1-minute window)
   - Compare against SLO thresholds
   - Trigger RCA if anomaly rate >1.2%

3. **Root Cause Analysis** (when triggered)
   - Extend analysis window (5s before + 55s after)
   - Partition traces into normal/anomalous sets
   - Build service dependency graphs from both sets
   - Calculate PageRank importance scores
   - Apply spectrum analysis to rank suspicious services

4. **Result Interpretation**
   - Higher scores = more likely root cause
   - Services appearing in many anomalous traces get higher scores
   - Services critical to system operation get weighted higher

## 🚫 Limitations

This is a **simplified simulation** for educational purposes:
- Uses synthetic data instead of real microservice traces
- Simulates time progression faster than real-time  
- Anomaly patterns are artificially generated
- No real service dependencies or network effects

For production use, you'll need:
- Real Elasticsearch deployment with Jaeger traces
- Proper time windows and thresholds tuned for your system
- Integration with monitoring and alerting systems

## 💡 Next Steps

1. **Run the demo** to see the workflow in action
2. **Experiment with parameters** (anomaly rates, thresholds, spectrum methods)
3. **Study the output** to understand how scores correlate with injected faults
4. **Adapt the mock data** to match your actual microservice architecture
5. **Deploy the real system** using the original MicroRank code with your Elasticsearch setup