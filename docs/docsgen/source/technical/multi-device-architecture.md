<!--
Copyright (c) ONNX Project Contributors

SPDX-License-Identifier: Apache-2.0
-->

# ONNX Multi-Device and Distributed Architecture

This document provides comprehensive architecture diagrams for ONNX multi-device execution, distributed inference, and parallel computation patterns.

## Multi-Device Execution Overview

ONNX supports distributed execution across multiple devices through tensor sharding, pipeline parallelism, and coordinated execution patterns.

```mermaid
graph TB
    subgraph MultiDeviceSystem["ONNX Multi-Device System"]
        subgraph ApplicationLayer["Application Layer"]
            ModelLoader["Model Loader<br/>Load multi-device model"]
            Orchestrator["Execution Orchestrator<br/>Coordinate distributed execution"]
            ResultAggregator["Result Aggregator<br/>Combine distributed outputs"]
        end
        
        subgraph ShardingSystem["Tensor Sharding System"]
            ShardingPlanner["Sharding Planner<br/>Plan tensor partitioning"]
            ShardDistributor["Shard Distributor<br/>Distribute tensor shards"]
            ShardCollector["Shard Collector<br/>Collect distributed results"]
            PartitionStrategy["Partition Strategy<br/>Splitting logic"]
        end
        
        subgraph DeviceCluster["Device Cluster"]
            Device0["Device 0<br/>Primary coordinator"]
            Device1["Device 1<br/>Worker node"]
            Device2["Device 2<br/>Worker node"]
            DeviceN["Device N<br/>Worker node"]
        end
        
        subgraph CommunicationLayer["Inter-Device Communication"]
            CollectiveOps["Collective Operations<br/>AllReduce, AllGather"]
            P2PComm["Point-to-Point<br/>Direct device transfer"]
            CommOptimizer["Communication Optimizer<br/>Bandwidth optimization"]
            SyncManager["Synchronization Manager<br/>Device coordination"]
        end
    end
    
    ModelLoader --> ShardingPlanner
    ShardingPlanner --> PartitionStrategy
    PartitionStrategy --> ShardDistributor
    ShardDistributor --> DeviceCluster
    
    Device0 <--> CommunicationLayer
    Device1 <--> CommunicationLayer
    Device2 <--> CommunicationLayer
    DeviceN <--> CommunicationLayer
    
    DeviceCluster --> ShardCollector
    ShardCollector --> ResultAggregator
    
    %% Styling
    classDef application fill:#e8f5e8
    classDef sharding fill:#e3f2fd
    classDef device fill:#fff3e0
    classDef communication fill:#f3e5f5
    
    class ModelLoader,Orchestrator,ResultAggregator,ApplicationLayer application
    class ShardingPlanner,ShardDistributor,ShardCollector,PartitionStrategy,ShardingSystem sharding
    class Device0,Device1,Device2,DeviceN,DeviceCluster device
    class CollectiveOps,P2PComm,CommOptimizer,SyncManager,CommunicationLayer communication
```

## Tensor Sharding Architecture

The tensor sharding system enables distributing large tensors across multiple devices efficiently.

```mermaid
graph TB
    subgraph TensorSharding["Tensor Sharding Architecture"]
        subgraph InputTensor["Input Tensor Processing"]
            OriginalTensor["Original Tensor<br/>Shape: [B, S, H]"]
            ShardingSpec["Sharding Specification<br/>Axis and device mapping"]
            ShardingValidator["Sharding Validator<br/>Validate shard compatibility"]
        end
        
        subgraph ShardingTypes["Sharding Types"]
            AxisSplitting["Axis Splitting<br/>Split along dimension"]
            Replication["Replication<br/>Copy to all devices"]
            PartialSharding["Partial Sharding<br/>Mixed strategy"]
        end
        
        subgraph ShardDistribution["Shard Distribution"]
            Device0Shard["Device 0<br/>Shard [0:B/2, S, H]"]
            Device1Shard["Device 1<br/>Shard [B/2:B, S, H]"]
            Device2Shard["Device 2<br/>Replicated [B, S, H]"]
            DeviceNShard["Device N<br/>Custom shard"]
        end
        
        subgraph ReassemblyLogic["Tensor Reassembly"]
            ShardTracker["Shard Tracker<br/>Track shard locations"]
            ConcatenationLogic["Concatenation Logic<br/>Reconstruct tensor"]
            ConsistencyChecker["Consistency Checker<br/>Validate reconstruction"]
        end
    end
    
    OriginalTensor --> ShardingSpec
    ShardingSpec --> ShardingValidator
    ShardingValidator --> ShardingTypes
    
    AxisSplitting --> Device0Shard
    AxisSplitting --> Device1Shard
    Replication --> Device2Shard
    PartialSharding --> DeviceNShard
    
    Device0Shard --> ShardTracker
    Device1Shard --> ShardTracker
    Device2Shard --> ShardTracker
    DeviceNShard --> ShardTracker
    
    ShardTracker --> ConcatenationLogic
    ConcatenationLogic --> ConsistencyChecker
    
    %% Styling
    classDef input fill:#e8f5e8
    classDef types fill:#e3f2fd
    classDef distribution fill:#fff3e0
    classDef reassembly fill:#f3e5f5
    
    class OriginalTensor,ShardingSpec,ShardingValidator,InputTensor input
    class AxisSplitting,Replication,PartialSharding,ShardingTypes types
    class Device0Shard,Device1Shard,Device2Shard,DeviceNShard,ShardDistribution distribution
    class ShardTracker,ConcatenationLogic,ConsistencyChecker,ReassemblyLogic reassembly
```

## Pipeline Parallelism Architecture

Pipeline parallelism enables distributing different parts of the model across devices for sequential execution.

```mermaid
graph LR
    subgraph PipelineParallelism["Pipeline Parallelism Architecture"]
        subgraph InputData["Input Data Flow"]
            BatchData["Input Batch<br/>Size: [B, ...]"]
            MicroBatches["Micro-batches<br/>Split for pipeline"]
        end
        
        subgraph PipelineStages["Pipeline Stages"]
            Stage0["Pipeline Stage 0<br/>Layers 0-3<br/>Device 0"]
            Stage1["Pipeline Stage 1<br/>Layers 4-7<br/>Device 1"]
            Stage2["Pipeline Stage 2<br/>Layers 8-11<br/>Device 2"]
            Stage3["Pipeline Stage 3<br/>Layers 12-15<br/>Device 3"]
        end
        
        subgraph PipelineControl["Pipeline Control"]
            Scheduler["Pipeline Scheduler<br/>Coordinate execution"]
            BufferManager["Buffer Manager<br/>Manage inter-stage data"]
            SyncController["Sync Controller<br/>Handle dependencies"]
        end
        
        subgraph OutputProcessing["Output Processing"]
            StageOutputs["Stage Outputs<br/>Intermediate results"]
            FinalAggregator["Final Aggregator<br/>Combine results"]
            OutputBuffer["Output Buffer<br/>Final results"]
        end
    end
    
    BatchData --> MicroBatches
    MicroBatches --> Stage0
    
    Stage0 --> Stage1
    Stage1 --> Stage2
    Stage2 --> Stage3
    
    Stage0 -.-> Scheduler
    Stage1 -.-> Scheduler
    Stage2 -.-> Scheduler
    Stage3 -.-> Scheduler
    
    Scheduler --> BufferManager
    BufferManager --> SyncController
    
    Stage3 --> StageOutputs
    StageOutputs --> FinalAggregator
    FinalAggregator --> OutputBuffer
    
    %% Styling
    classDef input fill:#e8f5e8
    classDef pipeline fill:#e3f2fd
    classDef control fill:#fff3e0
    classDef output fill:#f3e5f5
    
    class BatchData,MicroBatches,InputData input
    class Stage0,Stage1,Stage2,Stage3,PipelineStages pipeline
    class Scheduler,BufferManager,SyncController,PipelineControl control
    class StageOutputs,FinalAggregator,OutputBuffer,OutputProcessing output
```

## Communication Patterns

Inter-device communication patterns for distributed ONNX execution.

```mermaid
sequenceDiagram
    participant Coordinator as Coordinator Device
    participant Device1 as Worker Device 1
    participant Device2 as Worker Device 2
    participant Device3 as Worker Device 3
    participant CommLayer as Communication Layer

    Note over Coordinator,Device3: Model Distribution Phase
    Coordinator->>Device1: Send model shards
    Coordinator->>Device2: Send model shards
    Coordinator->>Device3: Send model shards
    
    Note over Coordinator,Device3: Input Distribution Phase
    Coordinator->>CommLayer: Distribute input tensor shards
    CommLayer->>Device1: Forward input shard 1
    CommLayer->>Device2: Forward input shard 2
    CommLayer->>Device3: Forward input shard 3
    
    Note over Coordinator,Device3: Parallel Execution Phase
    par Device 1 Execution
        Device1->>Device1: Execute local computation
    and Device 2 Execution
        Device2->>Device2: Execute local computation
    and Device 3 Execution
        Device3->>Device3: Execute local computation
    end
    
    Note over Coordinator,Device3: Intermediate Communication
    Device1->>CommLayer: AllReduce operation
    Device2->>CommLayer: AllReduce operation
    Device3->>CommLayer: AllReduce operation
    CommLayer->>Device1: Broadcast reduced result
    CommLayer->>Device2: Broadcast reduced result
    CommLayer->>Device3: Broadcast reduced result
    
    Note over Coordinator,Device3: Result Aggregation Phase
    Device1->>Coordinator: Send partial results
    Device2->>Coordinator: Send partial results
    Device3->>Coordinator: Send partial results
    
    Coordinator->>Coordinator: Aggregate final results
    
    Note over Coordinator: Final output ready
```

## Device Memory Management

Memory management strategies for multi-device ONNX execution.

```mermaid
graph TB
    subgraph MemoryManagement["Multi-Device Memory Management"]
        subgraph GlobalMemoryPool["Global Memory Pool"]
            SharedMemory["Shared Memory<br/>Cross-device accessible"]
            DeviceLocal["Device-Local Memory<br/>High-performance access"]
            HostMemory["Host Memory<br/>CPU-accessible buffer"]
        end
        
        subgraph MemoryCoordinator["Memory Coordinator"]
            AllocationPlanner["Allocation Planner<br/>Plan memory usage"]
            MemoryScheduler["Memory Scheduler<br/>Schedule transfers"]
            GarbageCollector["Garbage Collector<br/>Free unused memory"]
            MemoryProfiler["Memory Profiler<br/>Monitor usage patterns"]
        end
        
        subgraph DeviceMemory["Per-Device Memory"]
            Device0Memory["Device 0 Memory<br/>Local tensors & cache"]
            Device1Memory["Device 1 Memory<br/>Local tensors & cache"]
            Device2Memory["Device 2 Memory<br/>Local tensors & cache"]
            CommunicationBuffers["Communication Buffers<br/>Inter-device transfers"]
        end
        
        subgraph MemoryOptimizations["Memory Optimizations"]
            MemoryReuse["Memory Reuse<br/>Optimize allocations"]
            TensorFusion["Tensor Fusion<br/>Reduce fragmentation"]
            LazyLoading["Lazy Loading<br/>On-demand allocation"]
            CompressionEngine["Compression Engine<br/>Reduce transfer size"]
        end
    end
    
    AllocationPlanner --> SharedMemory
    AllocationPlanner --> DeviceLocal
    AllocationPlanner --> HostMemory
    
    MemoryScheduler --> Device0Memory
    MemoryScheduler --> Device1Memory
    MemoryScheduler --> Device2Memory
    MemoryScheduler --> CommunicationBuffers
    
    MemoryProfiler --> MemoryOptimizations
    GarbageCollector --> MemoryReuse
    
    Device0Memory <--> CommunicationBuffers
    Device1Memory <--> CommunicationBuffers
    Device2Memory <--> CommunicationBuffers
    
    %% Styling
    classDef global fill:#e8f5e8
    classDef coordinator fill:#e3f2fd
    classDef device fill:#fff3e0
    classDef optimization fill:#f3e5f5
    
    class SharedMemory,DeviceLocal,HostMemory,GlobalMemoryPool global
    class AllocationPlanner,MemoryScheduler,GarbageCollector,MemoryProfiler,MemoryCoordinator coordinator
    class Device0Memory,Device1Memory,Device2Memory,CommunicationBuffers,DeviceMemory device
    class MemoryReuse,TensorFusion,LazyLoading,CompressionEngine,MemoryOptimizations optimization
```

## Collective Operations Architecture

Implementation of collective communication operations for distributed ONNX execution.

```mermaid
graph TB
    subgraph CollectiveOps["Collective Operations Architecture"]
        subgraph AllReducePattern["AllReduce Pattern"]
            AllReduceInit["AllReduce Initiate<br/>Each device has partial data"]
            ReduceOperation["Reduce Operation<br/>Sum/Average/Max across devices"]
            BroadcastResult["Broadcast Result<br/>Share result to all devices"]
        end
        
        subgraph AllGatherPattern["AllGather Pattern"]
            GatherInit["AllGather Initiate<br/>Collect data from all devices"]
            ConcatenateOp["Concatenate Operation<br/>Combine collected data"]
            DistributeComplete["Distribute Complete<br/>Full dataset to all devices"]
        end
        
        subgraph ReduceScatterPattern["ReduceScatter Pattern"]
            ScatterInit["ReduceScatter Initiate<br/>Distribute and reduce"]
            PartialReduce["Partial Reduce<br/>Reduce assigned portions"]
            ScatterResult["Scatter Result<br/>Each device gets portion"]
        end
        
        subgraph CollectiveOptimization["Collective Optimization"]
            TopologyAware["Topology-Aware<br/>Optimize based on network"]
            HierarchicalReduce["Hierarchical Reduce<br/>Multi-level reduction"]
            OverlapCompute["Overlap Compute<br/>Hide communication latency"]
            AdaptiveAlgorithm["Adaptive Algorithm<br/>Choose best strategy"]
        end
    end
    
    AllReduceInit --> ReduceOperation
    ReduceOperation --> BroadcastResult
    
    GatherInit --> ConcatenateOp
    ConcatenateOp --> DistributeComplete
    
    ScatterInit --> PartialReduce
    PartialReduce --> ScatterResult
    
    TopologyAware --> HierarchicalReduce
    HierarchicalReduce --> OverlapCompute
    OverlapCompute --> AdaptiveAlgorithm
    
    %% Cross-connections for optimization
    AllReducePattern -.-> CollectiveOptimization
    AllGatherPattern -.-> CollectiveOptimization
    ReduceScatterPattern -.-> CollectiveOptimization
    
    %% Styling
    classDef allreduce fill:#e8f5e8
    classDef allgather fill:#e3f2fd
    classDef reducescatter fill:#fff3e0
    classDef optimization fill:#f3e5f5
    
    class AllReduceInit,ReduceOperation,BroadcastResult,AllReducePattern allreduce
    class GatherInit,ConcatenateOp,DistributeComplete,AllGatherPattern allgather
    class ScatterInit,PartialReduce,ScatterResult,ReduceScatterPattern reducescatter
    class TopologyAware,HierarchicalReduce,OverlapCompute,AdaptiveAlgorithm,CollectiveOptimization optimization
```

## Summary

This multi-device architecture documentation provides comprehensive coverage of:

1. **Multi-Device Execution**: Complete system for coordinated distributed processing
2. **Tensor Sharding**: Efficient strategies for distributing large tensors across devices
3. **Pipeline Parallelism**: Sequential processing across multiple devices for large models
4. **Communication Patterns**: Optimized inter-device communication protocols
5. **Memory Management**: Sophisticated memory coordination across devices
6. **Collective Operations**: Efficient implementation of distributed communication patterns

These diagrams enable developers to understand and implement distributed ONNX execution patterns, supporting large-scale model deployment and high-performance inference scenarios.