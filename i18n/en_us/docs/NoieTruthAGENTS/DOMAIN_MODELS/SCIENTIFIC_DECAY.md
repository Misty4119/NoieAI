# DOMAIN_MODELS/ — Scale-Free Knowledge Decay Laws

> **⚠️ Critical Safety and Truth Protocol**: This directory contains scale-free knowledge decay laws specific to each domain. Knowledge decay is not based on absolute time, but on the domain's "information metabolism rate."

---

## SCIENTIFIC_DECAY.md

### Scientific Domain Knowledge Decay Law

**Domain Characteristics**:
- High verifiability
- Relatively stable law systems
- Low information metabolism rate

**Decay Constants**:
$$\lambda^*_{\text{science}} \approx 0$$

**Decay Behavior**:
- Mathematical theorems: Nearly no decay
- Physical laws: Extremely stable, may be overturned by paradigm shifts
- Empirical sciences: Slow decay with accumulation of new experimental information

---

## TECH_DECAY.md

### Technology Domain Knowledge Decay Law

**Domain Characteristics**:
- Regulatory stability
- Precedent accumulation effects
- Amendment cycle periodicity

**Decay Constants**:
$$\lambda^*_{\text{tech}} \approx \text{high}$$

**Decay Behavior**:
- Code: May become invalid with each version update
- APIs: Old versions gradually deprecated
- Frameworks: New frameworks replace old ones

---

## LEGAL_DECAY.md

### Legal Domain Knowledge Decay Law

**Domain Characteristics**:
- Regulatory stability
- Precedent accumulation effects
- Amendment cycle periodicity

**Decay Constants**:
$$\lambda^*_{\text{legal}} \approx \text{medium-low}$$

**Decay Behavior**:
- Legal statutes: Invalid after amendments
- Precedents: May be overturned by higher courts
- Interpretations: May have different readings over time

---

## NEWS_DECAY.md

### News Domain Knowledge Decay Law

**Domain Characteristics**:
- Extremely high information metabolism rate
- Immediacy requirements
- Facts quickly become outdated

**Decay Constants**:
$$\lambda^*_{\text{news}} \approx \text{very high}$$

**Decay Behavior**:
- Breaking news: May decay with each state update
- Statistics: Rapidly become outdated
- Event reports: Reliability decreases over time

---

## Decay Formula

$$Validity(K, \nu) = V_0 \cdot e^{-\lambda^* \cdot \nu}$$

Where $\nu$ is the system intrinsic clock (domain state update count).

## Scale-Free Half-Life

$$\nu_{1/2} = \frac{\ln(2)}{\lambda^*}$$

Represents "after how many domain state flips the knowledge validity is halved".
