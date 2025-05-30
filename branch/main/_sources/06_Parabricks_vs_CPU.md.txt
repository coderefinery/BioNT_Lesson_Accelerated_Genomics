# Parabricks vs CPU-native NGS data processing

## Runtime comparison and speedup calculation

- Speedup = Runtime of base-process / Runtime of accelerated-process

| Step        | Base-runtime        | Accelerated-runtime | Speedup |
|-------------|---------------------|---------------------|---------|
| Mapping and <br> alignment refinement  | 1 h, 25 min, 14 sec      | 2 min, 39 sec       | 32.16   |
| GATK ApplyBQSR   | 29 min, 36 sec      | 34 sec              | 52.24   |
| GATK HaplotypeCaller | 1 h, 35 min, 5 sec  | 3 min, 13 sec       | 29.55   |

## Variant counts identified in CPU and Parabricks pipelines

## NAIC accelerated genomics pipeline

- [GitHub repository](https://github.com/NAICNO/accelerated_genomics)

### Our current activities


*Image sources: [NVIDIA](https://developer.nvidia.com/blog/nvidia-ampere-architecture-in-depth/) ; [DRAGEN](https://developer.illumina.com/dragen)*

- [NAIC LinkedIn page](https://www.linkedin.com/company/naic-norwegian-ai-cloud)