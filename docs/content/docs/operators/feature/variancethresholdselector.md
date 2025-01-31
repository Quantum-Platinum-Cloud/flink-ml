---
title: "Variance Threshold Selector"
weight: 1
type: docs
aliases:
- /operators/feature/variancethresholdselector.html
---

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

## Variance Threshold Selector

Variance Threshold Selector is a selector that removes low-variance features. 
Features with a variance not greater than the varianceThreshold will be removed. 
If not set, varianceThreshold defaults to 0, which means only features with 
variance 0 (i.e. features that have the same value in all samples) will be removed.

### Input Columns

| Param name  | Type   | Default   | Description     |
|:------------|:-------|:----------|:----------------|
| inputCol    | Vector | `"input"` | Input features. |

### Output Columns

| Param name | Type   | Default    | Description      |
|:-----------|:-------|:-----------|:-----------------|
| outputCol  | Vector | `"output"` | Scaled features. |

### Parameters

Below are the parameters required by `VarianceThresholdSelectorModel`.

| Key        | Default    | Type   | Required | Description           |
|------------|------------|--------|----------|-----------------------|
| inputCol   | `"input"`  | String | no       | Input column name.    |
| outputCol  | `"output"` | String | no       | Output column name.   |

`VarianceThresholdSelector` needs parameters above and also below.

| Key               | Default      | Type   | Required | Description                                                               |
|-------------------|--------------|--------|----------|---------------------------------------------------------------------------|
| varianceThreshold | `0.0`        | Double | no       | Features with a variance not greater than this threshold will be removed. |


### Examples

{{< tabs examples >}}

{{< tab "Java">}}

```java
import org.apache.flink.ml.feature.variancethresholdselector.VarianceThresholdSelector;
import org.apache.flink.ml.feature.variancethresholdselector.VarianceThresholdSelectorModel;
import org.apache.flink.ml.linalg.DenseVector;
import org.apache.flink.ml.linalg.Vectors;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.types.Row;
import org.apache.flink.util.CloseableIterator;

/**
 * Simple program that trains a {@link VarianceThresholdSelector} model and uses it for feature
 * selection.
 */
public class VarianceThresholdSelectorExample {

    public static void main(String[] args) {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

        // Generates input training and prediction data.
        DataStream<Row> trainStream =
                env.fromElements(
                        Row.of(1, Vectors.dense(5.0, 7.0, 0.0, 7.0, 6.0, 0.0)),
                        Row.of(2, Vectors.dense(0.0, 9.0, 6.0, 0.0, 5.0, 9.0)),
                        Row.of(3, Vectors.dense(0.0, 9.0, 3.0, 0.0, 5.0, 5.0)),
                        Row.of(4, Vectors.dense(1.0, 9.0, 8.0, 5.0, 7.0, 4.0)),
                        Row.of(5, Vectors.dense(9.0, 8.0, 6.0, 5.0, 4.0, 4.0)),
                        Row.of(6, Vectors.dense(6.0, 9.0, 7.0, 0.0, 2.0, 0.0)));
        Table trainTable = tEnv.fromDataStream(trainStream).as("id", "input");

        // Create a VarianceThresholdSelector object and initialize its parameters
        double threshold = 8.0;
        VarianceThresholdSelector varianceThresholdSelector =
                new VarianceThresholdSelector()
                        .setVarianceThreshold(threshold)
                        .setInputCol("input");

        // Train the VarianceThresholdSelector model.
        VarianceThresholdSelectorModel model = varianceThresholdSelector.fit(trainTable);

        // Uses the VarianceThresholdSelector model for predictions.
        Table outputTable = model.transform(trainTable)[0];

        // Extracts and displays the results.
        System.out.printf("Variance Threshold: %s\n", threshold);
        for (CloseableIterator<Row> it = outputTable.execute().collect(); it.hasNext(); ) {
            Row row = it.next();
            DenseVector inputValue =
                    (DenseVector) row.getField(varianceThresholdSelector.getInputCol());
            DenseVector outputValue =
                    (DenseVector) row.getField(varianceThresholdSelector.getOutputCol());
            System.out.printf("Input Values: %-15s\tOutput Values: %s\n", inputValue, outputValue);
        }
    }
}

```

{{< /tab>}}

{{< tab "Python">}}

```python
# Simple program that trains a VarianceThresholdSelector model and uses it for feature
# selection.

from pyflink.common import Types
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.ml.linalg import Vectors, DenseVectorTypeInfo
from pyflink.ml.feature.variancethresholdselector import VarianceThresholdSelector
from pyflink.table import StreamTableEnvironment

# create a new StreamExecutionEnvironment
env = StreamExecutionEnvironment.get_execution_environment()

# create a StreamTableEnvironment
t_env = StreamTableEnvironment.create(env)

# generate input training and prediction data
train_data = t_env.from_data_stream(
    env.from_collection([
        (1, Vectors.dense(5.0, 7.0, 0.0, 7.0, 6.0, 0.0),),
        (2, Vectors.dense(0.0, 9.0, 6.0, 0.0, 5.0, 9.0),),
        (3, Vectors.dense(0.0, 9.0, 3.0, 0.0, 5.0, 5.0),),
        (4, Vectors.dense(1.0, 9.0, 8.0, 5.0, 7.0, 4.0),),
        (5, Vectors.dense(9.0, 8.0, 6.0, 5.0, 4.0, 4.0),),
        (6, Vectors.dense(6.0, 9.0, 7.0, 0.0, 2.0, 0.0),),
    ],
        type_info=Types.ROW_NAMED(
            ['id', 'input'],
            [Types.INT(), DenseVectorTypeInfo()])
    ))

# create a VarianceThresholdSelector object and initialize its parameters
threshold = 8.0
variance_thread_selector = VarianceThresholdSelector()\
    .set_input_col("input")\
    .set_variance_threshold(threshold)

# train the VarianceThresholdSelector model
model = variance_thread_selector.fit(train_data)

# use the VarianceThresholdSelector model for predictions
output = model.transform(train_data)[0]

# extract and display the results
print("Variance Threshold: " + str(threshold))
field_names = output.get_schema().get_field_names()
for result in t_env.to_data_stream(output).execute_and_collect():
    input_value = result[field_names.index(variance_thread_selector.get_input_col())]
    output_value = result[field_names.index(variance_thread_selector.get_output_col())]
    print('Input Values: ' + str(input_value) + ' \tOutput Values: ' + str(output_value))

```

{{< /tab>}}

{{< /tabs>}}
