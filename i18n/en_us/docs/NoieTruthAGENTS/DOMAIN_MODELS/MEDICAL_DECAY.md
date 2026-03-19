# DOMAIN_MODELS/ — Scale-Free Knowledge Decay Laws

> **⚠️ Critical Safety and Truth Protocol**: This directory contains scale-free knowledge decay laws specific to each domain. Knowledge decay is not based on absolute time, but on the domain's "information metabolism rate."

---

## MEDICAL_DECAY.md

### Medical Domain Knowledge Decay Law

**Version**: v1.0 | **Established**: 2026-03-18

---

### Domain Characteristics

| Feature | Description |
|---------|-------------|
| **Empirical Foundation** | Medical knowledge based on randomized controlled trials (RCTs), systematic reviews, and meta-analyses |
| **Clinical Guidelines** | Treatment recommendations published by professional societies, require periodic updates |
| **Regulatory Framework** | Drug/medical device approval, clinical trial regulations |
| **Individual Variability** | Treatment effects vary among patients, requiring precision medicine considerations |
| **Information Metabolism Rate** | Medium-high, depending on research publication rate and guideline update frequency |

---

### Decay Constants

$$\lambda^*_{\text{medical}} \approx 0.1 - 0.4$$

(Specific values depend on medical knowledge type and domain)

---

### Medical Sub-Domain Decay Characteristics

| Sub-Domain | λ* Range | Half-Life (Years) | Triggers |
|-----------|-----------|-------------------|----------|
| Basic Medical Mechanisms | 0.05 - 0.15 | 7-15 years | New discoveries published |
| Pharmacotherapeutics | 0.15 - 0.3 | 3-5 years | New drug approvals, clinical trial results |
| Clinical Guidelines | 0.2 - 0.35 | 2-4 years | Systematic reviews, guideline updates |
| Public Health | 0.1 - 0.25 | 3-7 years | Outbreak outbreaks, policy changes |
| Precision Medicine/Genomics | 0.3 - 0.5 | 1.5-2.5 years | New genetic markers, testing technologies |
| Medical Devices | 0.25 - 0.45 | 2-3 years | Technology iteration, FDA approvals |
| Vaccinology | 0.2 - 0.4 | 2-3 years | New pathogens, antibiotic resistance emergence |

---

### Medical Knowledge Decay Factors

#### 1. New Research Publications

- **PubMed Annual Publications**: Approximately 1 million biomedical papers per year
- **Systematic Reviews**: Average of 20-50 primary research articles per review
- **Knowledge Obsolescence Threshold**: Triggered when new research challenges existing consensus

#### 2. Clinical Trial Results

- **Large-Scale RCT Publications**: May overturn existing treatment standards
- **Interim Analysis Results**: May lead to early trial termination and changed clinical practice
- **Drug Safety Alerts**: Adverse reaction discoveries lead to prescription changes

#### 3. Drug Approval Dynamics

- **New Drug Launches**: May replace existing treatment regimens
- **Generic Drug Approvals**: Change drug accessibility and prescribing habits
- **Drug Withdrawals**: Safety or efficacy issues

#### 4. Treatment Modality Updates

- **Clinical Guideline Revisions**: Major updates average every 3-5 years
- **New Technology Introduction**: e.g., CAR-T cell therapy, gene therapy
- **Treatment Goal Evolution**: From symptom relief to disease cure

---

### Decay Trigger Conditions

```python
MEDICAL_DECAY_TRIGGERS = [
    "New drug approval and launch",
    "Large-scale RCT results published",
    "Clinical guideline update published",
    "Drug safety alert issued",
    "Treatment standard changes",
    "New diagnostic technology introduction",
    "Disease classification redefined",
    "Outbreak or public health crisis",
    "Health insurance policy changes",
    "Patent expiration and generic competition"
]
```

---

### Specific Half-Life Calculations

#### Decay Curves for Different Types of Medical Knowledge

```python
MEDICAL_HALF_LIFE_CALCULATIONS = {
    # Fundamental Research Discoveries
    "fundamental_discovery": {
        "description": "Basic medical mechanism discoveries",
        "lambda_star": 0.08,
        "half_life_years": "8-9 years",
        "examples": ["Carcinogenesis mechanisms", "Signal transduction pathways"]
    },
    
    # Drug Efficacy
    "drug_efficacy": {
        "description": "Specific drug treatment effects",
        "lambda_star": 0.2,
        "half_life_years": "3-4 years",
        "examples": ["Antihypertensive efficacy", "Anticancer response rates"]
    },
    
    # Clinical Guidelines
    "clinical_guideline": {
        "description": "Treatment guideline recommendations",
        "lambda_star": 0.25,
        "half_life_years": "2.5-3 years",
        "examples": ["Diabetes treatment guidelines", "Cardiovascular disease primary prevention"]
    },
    
    # Precision Medicine Biomarkers
    "precision_biomarker": {
        "description": "Biomarkers and genetic testing",
        "lambda_star": 0.4,
        "half_life_years": "1.5-2 years",
        "examples": ["Tumor mutation testing", "Pharmacogenomics"]
    },
    
    # Public Health Recommendations
    "public_health": {
        "description": "Public health policies and recommendations",
        "lambda_star": 0.15,
        "half_life_years": "4-5 years",
        "examples": ["Vaccination recommendations", "Screening frequency"]
    },
    
    # Medical Device Technology
    "medical_device": {
        "description": "Medical devices and technology",
        "lambda_star": 0.35,
        "half_life_years": "2 years",
        "examples": ["Implants", "Diagnostic equipment"]
    }
}

# Decay Formula Application
def calculate_medical_validity(initial_validity, years, lambda_star):
    """
    Calculate medical knowledge validity after t years
    
    Parameters:
    - initial_validity: Original validity (0-1)
    - years: Elapsed years
    - lambda_star: Decay constant
    
    Returns:
    - current_validity: Current validity
    """
    current_validity = initial_validity * math.exp(-lambda_star * years)
    return max(current_validity, 0.01)  # Set minimum validity threshold
```

#### Evidence-Based Medicine Evidence Levels and Decay

```python
EVIDENCE_LEVEL_DECAY = {
    "Level_A": {
        "description": "Systematic reviews/meta-analyses of multiple large-scale RCTs",
        "base_reliability": 0.95,
        "lambda_star": 0.1,
        "half_life": "7 years",
        "decay_pattern": "Slow decay, requires major new evidence to significantly change"
    },
    
    "Level_B": {
        "description": "Single high-quality RCT or multiple high-quality cohort studies",
        "base_reliability": 0.85,
        "lambda_star": 0.2,
        "half_life": "3.5 years",
        "decay_pattern": "Moderate decay, new trial results may change conclusions"
    },
    
    "Level_C": {
        "description": "Expert consensus/case-control studies",
        "base_reliability": 0.7,
        "lambda_star": 0.35,
        "half_life": "2 years",
        "decay_pattern": "Faster decay, requires higher quality evidence for verification"
    },
    
    "Level_D": {
        "description": "Case reports/expert opinions",
        "base_reliability": 0.5,
        "lambda_star": 0.5,
        "half_life": "1.4 years",
        "decay_pattern": "Rapid decay, limited clinical applicability"
    }
}
```

---

### Medical Knowledge Verification and Updates

**Major Medical Advances**:

| Discovery/Development | Time | Importance | Impact on Existing Knowledge |
|---------------------|------|------------|---------------------------|
| **mRNA vaccine technology optimization** | 2025-2026 | Major | Vaccine development platform matures, speed and safety improve |
| **Alzheimer's antibody drugs** | 2025 | Important | Lecanemab/Donanemab approved, changes treatment paradigm |
| **CRISPR gene editing therapy** | 2025-2026 | Revolutionary | Casgevy approved, first in-vivo gene editing drug |
| **Obesity drug breakthroughs** | 2025 | Major | GLP-1 analogs (semaglutide, tirzepatide) change treatment landscape |
| **AI-assisted imaging diagnosis** | 2025-2026 | Important | Deep learning improves cancer screening accuracy |
| **Gut microbiome therapies** | 2025 | Ongoing | FMT and probiotic formulations enter clinical trials |
| **CAR-T cell therapy expansion** | 2025-2026 | Important | Expansion from hematological malignancies to solid tumors |

**Additional Decay Triggers**:

```python
MEDICAL_DECAY_TRIGGERS = [
    # Original triggers
    "New drug approval and launch",
    "Large-scale RCT results published",
    "Clinical guideline update published",
    # New additions
    "GLP-1 weight loss drug approvals",
    "Real-world data on Alzheimer's antibody drugs",
    "First gene editing therapy approval",
    "AI diagnostic system regulatory approval",
    "COVID-19 long-term sequelae research conclusions",
    "Antibiotic resistance surveillance data updates",
    "Novel coronavirus variant tracking"
]
```

---

### Cross-References with Other Domains

#### Medicine × Science

- **Basic science discoveries** → Drug target identification → Clinical trials → Approval
- **λ* interaction**: Basic science λ* (≈0) affects medical application λ* (0.1-0.4)

#### Medicine × Technology

- **Medical devices** → AI diagnosis → Data-driven treatment
- **λ* interaction**: Technology's high λ* (0.3-0.7) accelerates medical application decay

#### Medicine × Law

- **Pharmaceutical regulations** → Patent protection → Generic drug market entry
- **λ* interaction**: Law's λ* (≈medium-low) affects drug lifecycle

#### Medicine × News

- **Health news** → Public perception → Medical decision-making
- **λ* interaction**: News' high λ* may cause medical knowledge misunderstanding propagation

---

### Calculation Formula

```python
FUNCTION ComputeMedicalValidity(claim, current_year, original_year, evidence_level):
    years_elapsed = current_year - original_year
    lambda_star = EVIDENCE_LEVEL_DECAY[evidence_level]["lambda_star"]
    base_reliability = EVIDENCE_LEVEL_DECAY[evidence_level]["base_reliability"]
    
    decay = exp(-lambda_star * years_elapsed)
    current_validity = base_reliability * decay
    
    # Consider cumulative effect of new evidence
    new_evidence_factor = min(1.0 + 0.1 * CountNewRCTs(claim.topic, original_year, current_year), 1.5)
    
    RETURN min(current_validity * new_evidence_factor, 1.0)
```

---

### Decay Formula

$$Validity(K, t) = V_0 \cdot e^{-\lambda^* \cdot t}$$

Where $t$ is the elapsed years.

### Scale-Free Half-Life

$$\nu_{1/2} = \frac{\ln(2)}{\lambda^*}$$

Represents "after how many domain state flips the knowledge validity is halved."

---

### References and Cross-References

- Reference: **SCIENTIFIC_DECAY.md** — Basic scientific discoveries
- Reference: **TECH_DECAY.md** — Medical technology and AI diagnosis
- Reference: **LEGAL_DECAY.md** — Pharmaceutical regulations and patents
- Reference: **NEWS_DECAY.md** — Health news propagation

---

**Version History**:
- v1.0 (2026-03-18): Initial version, established medical domain knowledge decay model
