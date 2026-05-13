# Data and Data Pipelines Diagrams Description

**Main Source document:** Data_info_DD.ods  
**Purpose of the diagrams:** Define organizational structure for medical imaging data and processing pipelines

## 1. Overview

The DATA folder requires a two-level abstraction to organize both the **raw data assets** and the **processing workflows** that operate on them.

### Goal
Establish two abstraction levels:
1. **Data** — taxonomy of raw data assets (what data do we have)
2. **Data pipelines** — taxonomy of processing workflows (what do we do with the data)

---

## 2. "Data" diagram
**Purpose:** Classify and locate all data assets independent of how they're used.

### 2.1 Taxonomy Structure

**Level 1: Data**
- Purpose: Root entry point for all data assets

**Level 2: Type of collaboration**
- Purpose: Classification by data acquisition partnership

**Level 3: The source of the data**
- Purpose: Identification of specific data provider

**Level 4: Type of data**
- Purpose: Classification of content structure

**Level 5: The modality of the data**
- Purpose: Specification of imaging technique

**Level 5b (Conditional): Method of data creation**
- Purpose: Specification of how data were created/prepared **prior to storage**

  *Note: The Ground Truth (GT) folder name refers to data prepared directly by the hospital. For public datasets, GT labels are the annotations provided with the dataset, regardless of whether they were created manually or automatically.*

**Level 6: Organ name**
- Purpose: Specification of anatomical coverage(ROI)

**Level 7: Data**
- Purpose: Concrete data with related prefixes

#### Prefixes taxonomy:
  1. Annotation files: RTSTRUCT (RTS, STR, RS, RT);
  2. Main raw images: CT; MRI (MR, NMR); CBCT;
  3. RTIMAGE;
  4. Dosage files: RT Dose, RD;
  5. CTSCOUT
  6. GT registration files: RG, REGR;
  7. DIR files: DR;
  8. Planning files: RT Plans, RP;

*Note: The prefixes depend on the source of the data and they are not a rule; they are informal.*


---

#### 2.2 Type of data (Level 4)
In the diagram there is a division by the type of data (Level 4). 
There are different types of data which we use in our pipelines:
1. Raw medical images;
2. Annotation files: RTSTRUCT files;
3. Paired files: images are paired when they are acquired at the same time for different modalities in such a way that every voxel from the first image corresponds to a voxel from the second image;
4. Registration files: images are registered when they are acquired at different times for different modalities in such a way that every voxel from the first image corresponds to a voxel from the second image. However, they are not paired in the real sense;
5. Linked files: images are linked if there are any characteristics of relationship between them (patient ID as an example);
    
    *Note: The term is not official; it is our internal term.*
6. Dosage files (in progress);
7. Planning files: files which are based on the first planning stage results from planning CT;
## 3. Diagram "Data pipelines"

**Purpose:** Document processing workflows and their data dependencies.

**Usage:** When there is a need to create a dataset, a person can create a folder which will consist of links to the original data.
### 3.1 Taxonomy Structure

**Level 1: Data pipelines**
- Purpose: Root entry point for all processing workflows.

**Level 2: Processing Domain**
- Purpose: High-level classification of task type.

**Level 3: Method/Model**
- Purpose: Specification of approach or subtask.

**Level 4: Method stage details**
- Purpose: Details about the approach.

**Level 5: DataReference**
- Purpose: Concrete data source reference (link).
  Note: At this level there might be a need to add additional info about the data (like configs) or experiment results.

---
