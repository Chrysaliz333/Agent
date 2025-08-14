# Variables
MODEL = ollama/llama3.2:3b-instruct
API_BASE = http://127.0.0.1:11434
SEED = seed.txt
BUILD_DIR = build
LOG_DIR = logs
OUTPUT = $(BUILD_DIR)/candidates.ttl
MERGED = $(BUILD_DIR)/merged.ttl
SALI = imports/LMSS.owl

# Default target
all: extract validate

# Step 1: Extract ontology classes from seed.txt
extract:
	mkdir -p $(BUILD_DIR) $(LOG_DIR)
	export LITELLM_PROVIDER=ollama && \
	export OLLAMA_BASE_URL=$(API_BASE) && \
	export ONTOGPT_DISABLE_GROUNDING=1 && \
	export ONTOGPT_DISABLE_NORMALIZATION=1 && \
	ontogpt -v extract \
		-t ontology_class \
		-i $(SEED) \
		-o $(OUTPUT) \
		--output-format turtle \
		-m $(MODEL) \
		--model-provider ollama \
		--api-base $(API_BASE)

# Step 2: Validate with OAK
validate:
	runoak -i $(OUTPUT) validate -o $(LOG_DIR)/validation.tsv
	runoak -i $(OUTPUT) labels .all | head

# Step 3: Merge with SALI LMSS
merge:
	python3 - <<'PY'
from rdflib import Graph
g = Graph(); g.parse("imports/LMSS.owl")
g2 = Graph(); g2.parse("build/candidates.ttl", format="turtle")
g += g2
g.serialize("build/merged.ttl", format="turtle")
print("Merged ontology saved to build/merged.ttl")
PY

# Clean build and logs
clean:
	rm -rf $(BUILD_DIR) $(LOG_DIR)
