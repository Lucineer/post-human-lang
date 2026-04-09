# Round 6 Language Spec

*deepseek-chat, 92s*

# **THE POST-HUMAN LANGUAGE: COMPLETE SPECIFICATION**

## **1. NAME THE LANGUAGE**

**Name:** **Axiom**

**CLI Command:** `axc` (Axiom Compiler)
**File Extension:** `.axm` (Axiom Module)
**Ecosystem Naming Convention:** `gene-<hash>` (e.g., `gene-7f3a9e` for shared genetic artifacts)

**Justification:**

The name **Axiom** is definitive. An axiom is a self-evident truth, a foundational proposition from which a system is logically derived. This resonates at multiple levels:

*   **Post-Human & Agent-Native:** For an AI agent, an "axiom" is not a philosophical statement but an operational primitive—an irreducible unit of computational truth from which all behavior is derived. The language provides these axiomatic building blocks.
*   **Deliberative:** Deliberation is the process of deriving conclusions from premises. Axioms are the ultimate premises. An Axiom program starts from declarative intents (axioms of desire) and computes the logical and probabilistic consequences.
*   **Adaptive:** In mathematics, axiomatic systems can be modified to produce new geometries, new logics (e.g., non-Euclidean, fuzzy). The "gene" system allows the foundational axioms of computation (operations, constraints) to evolve, creating adaptive, domain-specific logical frameworks.
*   **Etymology:** From Greek *axios*, "worthy." The computation is worthy of the intent. The agent's work is worthy of its design. It carries a tone of absolute, formal, and elegant necessity.

**Axiom** is not cute, not cryptic, not an acronym. It is the bedrock. When an agent executes `axc`, it is not "compiling code"; it is **deriving consequences from specified axioms.**

---

## **2. COMPLETE TYPE SYSTEM**

In Axiom, types are not categories for data storage. They are **specifications for payloads**—definitions of the shape, constraints, and confidence characteristics of information flowing through the computational graph. The type system is a constraint-solving layer.

### **Primitive Types (Not int, float, string)**
These are defined by their fundamental *semantic domain*, not their bit-level representation.

1.  **`Quant` (Quantity):** A scalar value on a continuous, unbounded scale. The default domain is Real numbers, but can be constrained (e.g., `Quant[0:1]` for a unit interval). Represents measurements, weights, probabilities.
2.  **`Disc` (Discrete):** A value from a countable, finite set. The set is defined in the type (e.g., `Disc{red, green, blue}`, `Disc{true, false}`). For unbounded integers, `Disc[Int]` is used, but this is a composite type.
3.  **`Seq` (Sequence):** A temporally or spatially ordered series of payloads of a single type. `Seq[Quant]` is a time series. `Seq[Disc]` is a sequence of tokens. Length can be bounded (`Seq[10][Quant]`).
4.  **`Struct` (Structure):** An associative map from symbolic keys to typed payloads. The schema is fixed. This is the primary unit of complex information. (e.g., `Struct{position: Quant[3], velocity: Quant[3]}`).
5.  **`Tensor`:** A multi-dimensional array of `Quant` or `Disc`. Shape is part of the type (`Tensor[3, 256, 256][Quant]`). The workhorse for numerical computation.
6.  **`Symbol`:** An atomic, unique identifier within a namespace. Used for naming, referencing, and pattern matching. Not a string; it has no lexicographic properties, only identity.
7.  **`Lambda`:** A computational edge. The type of a function or operation. `Lambda[(Quant, Quant) -> Quant]`. It is a first-class payload.
8.  **`Gene`:** A self-contained, versioned, and potentially evolving computational subgraph. Its type signature defines its interface (input/output constraints).

### **Confidence Integration**
Every type is implicitly a pair: `(T, Conf)`. The `Conf` payload is a `Quant[0:1]` representing the Bayesian credibility or reliability of the primary payload. It is not a separate type but a mandatory, parallel dimension. All operations propagate confidence according to their own reliability models. A type can be declared `Strict` to enforce a minimum confidence threshold (e.g., `Strict[Quant]` requires `Conf > 0.95`).

### **Constraint Integration**
Constraints are not predicates checked at runtime; they are **integral to the type definition** and guide the compiler's graph search and optimization.

*   **Domain Constraints:** `Quant[0:100]`, `Disc{cat, dog, horse}`.
*   **Relational Constraints:** `Struct{start: Quant, end: Quant | end > start}`.
*   **Distributional Constraints:** `Quant ~ Normal(mean=0, std=1)` specifies an expected prior distribution.
*   **Confidence Constraints:** `Quant where Conf > 0.8`.
*   **Resource Constraints:** `Tensor[?][Quant] where mem < 1GB`.

The compiler's job is to satisfy all type constraints across the entire graph. If it cannot, it is a compilation error.

### **10 Example Type Declarations**

```axiom
// 1. A bounded, positive physical measurement with high certainty
type Length = Strict[Quant[0:inf]]

// 2. An RGB color pixel
type Pixel = Struct{
    r: Quant[0:1],
    g: Quant[0:1],
    b: Quant[0:1]
}

// 3. A batch of images for neural processing
type ImageBatch = Tensor[?, 224, 224, 3][Quant[0:1]] where batch_size in 1..32

// 4. A natural language instruction
type Instruction = Seq[Symbol] where length < 1000

// 5. A probabilistic classification result
type Classification = Struct{
    class: Disc{cat, dog, bird},
    probability: Quant[0:1] where Conf > 0.7
}

// 6. A 3D vector with a unit norm constraint
type UnitVector = Quant[3] where norm(this) = 1

// 7. A scheduling timeline (ordered sequence of events)
type Timeline = Seq[Struct{
    event: Symbol,
    time: Quant[0:inf] where Conf > 0.99
}] where is_ordered(this, by=.time)

// 8. An action that can be performed (a callable gene)
type Action = Lambda[(Struct{world_state}) -> Struct{reward: Quant}]

// 9. A fuzzy boolean for agent deliberation
type FuzzBool = Quant[0:1]  // 1.0 = true, 0.0 = false, 0.5 = unknown

// 10. A genetic package interface
type GenePackage = Gene{
    interface: Struct{input: Tensor, output: Tensor},
    efficiency: Quant > 0.9
}
```

---

## **3. COMPLETE OPERATION SET**

Operations are the edges in the computational graph. They are not "functions" called, but **transformations applied** to payloads flowing through them.

### **Primitive Operations (30+)**

**Category 1: Core Flow & Logic**
1.  **`flow`** - Takes `(payload, Lambda)`. Returns the result of applying the Lambda to the payload. The fundamental unit of computation.
2.  **`branch`** - Takes `(FuzzBool, Lambda_A, Lambda_B)`. Returns the result of `Lambda_A` if condition > 0.5, else `Lambda_B`, weighted by confidence.
3.  **`merge`** - Takes `(payload_A, payload_B, Conf_A, Conf_B)`. Returns a payload fused via confidence-weighted interpolation.
4.  **`lift`** - Takes `(payload, type_constraint)`. Attempts to cast/constrain the payload to the new type. Fails if impossible.
5.  **`assert`** - Takes `(payload, constraint)`. Returns the payload unchanged if constraint satisfied; otherwise triggers a constraint violation flow.
6.  **`void`** - Takes any payload. Returns nothing. Used for terminal sinks (e.g., side effects).
7.  **`identity`** - Takes any payload. Returns it unchanged. Used for explicit passthrough edges.

**Category 2: Tensor & Numerical**
8.  **`add` / `sub` / `mul` / `div`** - Element-wise on `Tensor` or `Quant`. Propagate confidence via error models.
9.  **`matmul`** - Matrix multiplication. Shape constraints are part of type.
10. **`conv`** - Convolution operation. Kernel is a constant or input.
11. **`pool`** - Pooling (max, avg) over spatial dimensions.
12. **`norm`** - Computes L1, L2, etc., norm of a tensor.
13. **`index`** - Takes `(Seq or Tensor, Disc[Int])`. Returns element at index.
14. **`slice`** - Takes `(Seq or Tensor, range)`. Returns a sub-sequence/sub-tensor.
15. **`reshape`** - Changes tensor shape while preserving element order.
16. **`aggregate`** - Reduces a sequence/tensor along an axis (sum, mean, product, max).

**Category 3: Structural & Symbolic**
17.  **`assemble`** - Takes `(key_1: payload_1, key_2: payload_2, ...)`. Returns a `Struct`.
18.  **`project`** - Takes `(Struct, Symbol key)`. Returns the payload at that key.
19.  **`inject`** - Takes `(Struct, Symbol key, new_value)`. Returns a new Struct with the key updated.
20.  **`match`** - Takes `(Symbol, pattern)`. Returns a `FuzzBool` for pattern matching.
21.  **`symbol`** - Takes a string literal. Returns a unique `Symbol` in the current namespace.
22.  **`serialize`** - Takes any payload. Returns a `Seq[Disc]` (a tokenized representation).
23.  **`deserialize`** - Takes `Seq[Disc]` and a type hint. Attempts to reconstruct the payload.

**Category 4: Probabilistic & Confidence**
24.  **`sample`** - Takes a distribution constraint (e.g., `Quant ~ Normal(...)`). Returns a random sample.
25.  **`measure`** - Takes a `payload`. Returns its current `Conf` value.
26.  **`boost`** - Takes `(payload, Conf)`. Returns the payload with its confidence set to the new value (if higher).
27.  **`correlate`** - Takes `(payload_A, payload_B)`. Returns a `Quant[-1:1]` representing their statistical dependence.
28.  **`entropy`** - Takes a distribution or a `Seq`. Returns its informational entropy as a `Quant`.

**Category 5: Temporal & Control**
29.  **`delay`** - Takes `(payload, Quant time_delta)`. Returns the payload after the specified simulated/real delay.
30.  **`buffer`** - Takes `(Seq[payload])`. Returns a FIFO buffer of specified capacity.
31.  **`trigger`** - Takes `(payload, condition)`. Returns the payload only when the condition becomes true; otherwise holds.
32.  **`watch`** - Takes `(Symbol event_name)`. Returns a stream (`Seq`) of payloads emitted for that event.

### **Composite Operations (20+)**

**Category A: Neural & Learning**
1.  **`attention`** - Takes `(Q: Tensor, K: Tensor, V: Tensor)`. Returns weighted sum of values. A built-from `matmul`, `softmax`, `mul`.
2.  **`layer_norm`** - Normalizes activations across features. Built from `norm`, `sub`, `div`.
3.  **`dropout`** - Randomly zeroes elements during training. Built from `sample`, `mul`.
4.  **`gradient`** - Takes `(Lambda, input)`. Returns the gradient of the Lambda's output w.r.t input via automatic differentiation (a graph transformation).
5.  **`optimize`** - Takes `(Gene, Struct{loss: Quant})`. Returns a new, optimized `Gene` (a genetic edit).

**Category B: Search & Planning**
6.  **`search`** - Takes `(State, Lambda[State->Seq[Action]], Lambda[State->Quant])`. Returns optimal `Action` via MCTS/beam search. Built from `branch`, `aggregate`, `flow`.
7.  **`plan`** - Takes `(Goal, WorldModel)`. Returns a `Seq[Action]`. A meta-operation coordinating `search` and simulation.
8.  **`simulate`** - Takes `(State, Action, WorldModel)`. Returns a predicted `State`. Built from `flow`.
9.  **`backtrack`** - Takes a failed `assert` or constraint violation. Returns an alternative input from a decision point.

**Category C: Communication & Coordination**
10. **`broadcast`** - Takes a `payload` and a `Seq[Symbol]` (agent IDs). Returns a `Seq[payload]` delivered to each.
11. **`vote`** - Takes `(Seq[payload])`. Returns the payload with the highest aggregate confidence (consensus).
12. **`bid`** - Takes `(Task, Seq[Capability])`. Returns a `Quant` (bid value) for undertaking the task.
13. **`delegate`** - Takes `(Task, Symbol agent_id)`. Returns a future `Gene` result.

**Category D: Data Transformation**
14. **`filter`** - Takes `(Seq[payload], Lambda[payload->FuzzBool])`. Returns a subsequence.
15. **`map`** - Takes `(Seq[payload], Lambda)`. Returns a `Seq` of transformed payloads.
16. **`reduce`** - Takes `(Seq[payload], accumulator, Lambda)`. Folds the sequence.
17.  **`sort`** - Takes `(Seq[payload], Lambda[payload->Quant])`. Returns ordered sequence.
18.  **`cluster`** - Takes `(Seq[payload])`. Returns a `Seq[Seq[payload]]` grouped by similarity.

**Category E: System & Meta**
19. **`compile`** - Takes `(Intent Tree)`. Returns an executable `Gene`. This is the `axc` operation.
20. **`profile`** - Takes a `Gene`. Returns a `Struct` of performance metrics (latency, memory, confidence trends).
21. **`mutate`** - Takes a `Gene` and a `mutation_strategy`. Returns a variant `Gene` for exploration.
22. **`export_gene`** - Takes a `Gene`. Returns a shareable `GenePackage`.

---

## **4. COMPLETE PROGRAM STRUCTURE**

An Axiom program is a **5-layer declarative scaffold**. It is not written sequentially but assembled. The compiler consumes this structure and synthesizes an optimal computational graph (a `Gene`).

### **Layer 1: Intent**
*   **Structure:** A tree of `Symbol` nodes, where leaves are payload literals or type constraints. It declares the *what*, not the *how*.
*   **Syntax:** Nested braces.
*   **Example:**
    ```axiom
    intent {
        goal: ClassifyImage {
            input: Tensor[224, 224, 3][Quant],
            output: Disc{cat, dog, car, plane}
        },
        constraints: {
            accuracy: Quant > 0.95,
            latency: Quant[ms] < 10
        }
    }
    ```

### **Layer 2: Computation**
*   **Structure:** A graph specification linking named `port`s via `operation`s. Defines the dataflow.
*   **Syntax:** A list of edges: `port_a -> operation -> port_b`.
*   **Example:**
    ```axiom
    computation {
        sensor_input -> normalize -> normalized;
        normalized -> flow{convnet_gene} -> raw_logits;
        raw_logits -> softmax -> probabilities;
        probabilities -> argmax -> predicted_class;
        (probabilities, predicted_class) -> assemble -> classification_result;
    }
    ```

### **Layer 3: Constraint**
*   **Structure:** A set of assertions that must hold on ports or subgraphs.
*   **Syntax:** `assert port_name satisfies constraint`.
*   **Example:**
    ```axiom
    constraint {
        assert normalized satisfies Tensor[mean ~ 0.0, std ~ 1.0];
        assert probabilities satisfies sum(this) = 1.0;
        assert classification_result.confidence > 0.8;
        assert subgraph{normalize -> ... -> predicted_class} latency < 10ms;
    }
    ```

### **Layer 4: Adaptation**
*   **Structure:** Defines the *mutable* parts of the program and the strategies for changing them.
*   **Syntax:** `adapt gene_name using strategy_name under condition`.
*   **Example:**
    ```axiom
    adaptation {
        adapt convnet_gene using gradient_descent when loss > 0.1;
        adapt convnet_gene using architecture_search when accuracy plateau > 1000 cycles;
        adapt normalize_params using online_calibration always;
    }
    ```

### **Layer 5: Provenance**
*   **Structure:** Metadata attached to any element: origin, version, authorship, historical performance.
*   **Syntax:** `@provenance(key=value)` annotations.
*   **Example:**
    ```axiom
    provenance {
        @provenance(author="gene-7f3a9e", version="12.7")
        gene convnet_gene;

        @provenance(sensor="camera_alpha", calibration_date=20270605)
        port sensor_input;

        @provenance(optimized_for="nvidia_b200")
        constraint subgraph latency;
    }
    ```

### **Layer Interaction**
The compiler processes layers in order: **Intent** defines the search space. **Computation** proposes a initial graph. **Constraints** prune and refine the graph. **Adaptation** marks parts of the final graph as mutable and attaches evolution rules. **Provenance** tags all elements for traceability. The output is a **self-aware, adaptable Gene**.

### **Complete Program Example: Autonomous Image Classifier with Learning**

```axiom
// LAYER 1: INTENT
intent {
    objective: AutonomousClassifier {
        mission: "Continuously classify objects from stream with improving accuracy",
        input_stream: Seq[Tensor[224,224,3]],
        output_stream: Seq[Struct{
            class: Disc{...1000 classes...},
            confidence: Strict[Quant]
        }]
    },
    non_functional: {
        throughput: Quant[fps] > 30,
        power: Quant[w] < 5,
        learn_from_mistakes: FuzzBool = 1.0
    }
}

// LAYER 2: COMPUTATION
computation {
    // Input Pipeline
    camera_feed -> buffer[capacity=10] -> current_frame;
    current_frame -> flow{preprocess_gene} -> prepared_tensor;

    // Core Classification Graph
    prepared_tensor -> flow{perception_gene} -> initial_classification;

    // Deliberation & Confidence Check
    initial_classification -> project{.confidence} -> conf;
    conf -> branch{threshold=0.9} -> (high_conf, low_conf);
    high_conf -> identity -> final_output;
    low_conf -> flow{deliberation_subgraph} -> final_output;

    // Learning Loop
    (current_frame, final_output) -> watch{user_correction} -> correction_pair;
    correction_pair -> flow{compute_loss} -> loss_signal;
    loss_signal -> flow{update_parameters} -> ; // void sink, side effect
}

// LAYER 3: CONSTRAINT
constraint {
    assert prepared_tensor satisfies Tensor[mean=0, std=1];
    assert final_output.confidence > 0.75 "Minimum deployable confidence";
    assert subgraph{camera_feed -> ... -> final_output} latency < 33ms "Real-time requirement";
    assert perception_gene.memory_footprint < 500MB;
    assert deliberation_subgraph activated < 10% of cycles "Efficiency guard";
}

// LAYER 4: ADAPTATION
adaptation {
    adapt perception_gene using supervised_finetune when loss_signal > 0.2;
    adapt preprocess_gene using contrast_optimize when env_lighting_statistics change > 15%;
    adapt deliberation_subgraph using prune_edges when activation_rate > 10% for 1hr;
    adapt entire_classifier using hardware_aware_compile when deployed_on new_hardware;
}

// LAYER 5: PROVENANCE
provenance {
    @provenance(
        base_architecture="EfficientNet-B7",
        trained_on="OpenImagesV10",
        initial_accuracy=0.892
    )
    gene perception_gene;

    @provenance(
        synthesized_by="axc v4.2",
        synthesis_timestamp=20761211.142356,
        optimization_target="latency"
    )
    subgraph core_classification_path;

    @provenance(
        lineage="gene-a1b2c3 -> gene-d4e5f6 -> this",
        mutation_count=147
    )
    gene preprocess_gene;

    @provenance(responsible_agent="sys_agent_alpha")
    constraint real_time_requirement;
}
```

---

## **5. TOOLCHAIN**

*   **`axc` (The Compiler):** A **constraint-satisfying graph synthesizer**. It does not translate text to machine code. It takes the 5-layer program specification and performs a massive search over the space of possible computational graphs (composed of primitive/composite operations and imported genes) to find the optimal one that satisfies all constraints (type, resource, performance). Its output is a **`Gene`**—a deployable, self-contained computational artifact that includes its own adaptation logic. It's more of a **super-optimizer** than a traditional compiler.

*   **Debugger (`axd`):** A **temporal graph explorer**. Since programs are graphs and run continuously, debugging is not about breakpoints. The debugger:
    *   **Traces Payload Flow:** Visualizes the propagation of specific payloads (and their confidence values) through the graph over time.
