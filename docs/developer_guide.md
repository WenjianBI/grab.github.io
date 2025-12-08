---
layout: default
title: Developer Guide for Method Integration
nav_order: 6
---

**Overview:** GRAB uses a two-step framework: `GRAB.NullModel()` fits a null model, then `GRAB.Marker()` performs association tests. To integrate a new method called `MyMethod`, you need to:

1. Create **six R functions** in `R/MyMethod.R`
2. Create **one C++ class** in `src/MyMethod.h` and `src/MyMethod.cpp`
3. **Register** the functions in `R/GRAB_Null_Model.R`, `R/GRAB_Marker.R`, and `src/Main.cpp`

Below we first present:

1. [Workflow with Registration Examples](#1-workflow-with-registration-examples)
2. [Examples for Each Function](#2-examples-for-each-function)

---

## 1. Workflow with Registration Examples

### 1.1 `GRAB.NullModel`

Example call:

```r
obj.null <- GRAB.NullModel(pheno ~ covariates, method = "MyMethod", ...)
```

Create 3 R functions in `R/MyMethod.R`:

- [GRAB.MyMethod](#21-grabmymethod)
- [checkControl.NullModel.MyMethod](#22-checkcontrolnullmodelmymethod)
- [fitNullModel.MyMethod](#23-fitnullmodelmymethod)

Register in `GRAB.NullModel`:

```r
# 1. Add method name to supported list
supported_methods <- c("POLMM", ..., "MyMethod")

# 2. Validate common parameters first, then method-specific parameters
checkResult <- switch(method,
  POLMM = checkControl.NullModel.POLMM(...),
  ...,
  MyMethod = checkControl.NullModel.MyMethod(traitType, ...)
)

# 3. Fit null model using method-specific function
objNull <- switch(method,
  POLMM = fitNullModel.POLMM(...),
  ...,
  MyMethod = fitNullModel.MyMethod(response, designMat, ...)
)
```

---

### 1.2 `GRAB.Marker`

Example call:

```r
GRAB.Marker(obj.null, GenoFile, OutputFile, ...)
```

Create 3 R functions in `R/MyMethod.R`:

- [checkControl.Marker.MyMethod](#24-checkcontrolmarkermymethod)
- [setMarker.MyMethod](#25-setmarkermymethod)
- [mainMarker.MyMethod](#26-mainmarkermymethod)

Register in `GRAB.Marker`:

```r
# 1. Add null model class name to supported list
supported_classes <- c(
  "POLMM_NULL_Model",
  ...,
  "MyMethod_NULL_Model"
)

# 2. Validate marker-level control parameters
control <- switch(NullModelClass,
  POLMM_NULL_Model = checkControl.Marker.POLMM(control),
  ...,
  MyMethod_NULL_Model = checkControl.Marker.MyMethod(control)
)

# 3. Initialize C++ backend with null model objects
switch(NullModelClass,
  POLMM_NULL_Model = setMarker.POLMM(objNull, control),
  ...,
  MyMethod_NULL_Model = setMarker.MyMethod(objNull, control)
)

# 4. Loop over genotype chunks, perform tests, format output
resMarker <- switch(NullModelClass,
  POLMM_NULL_Model = mainMarker.POLMM(genoType, genoIndex),
  ...,
  MyMethod_NULL_Model = mainMarker.MyMethod(genoType, genoIndex)
)
```

---

### 1.3 `src/Main.cpp`

Entry points within `GRAB.Marker`:

```r
setMarker.MyMethod(objNull, control)
mainMarker.MyMethod(genoType, genoIndex)
```

Define a C++ class `MyMethodClass` in `src/MyMethod.cpp` and `src/MyMethod.h` within the `MyMethod` namespace, implementing the following functions:

- [MyMethodClass](#27-mymethodclass)
- [getAnyResult](#27-mymethodclass)

Register in `src/Main.cpp`:

```cpp
// 1. Include header file
#include "MyMethod.h"

// 2. Declare global object pointer
static MyMethod::MyMethodClass* ptr_gMyMethodObj = nullptr;

// 3. Function to initialize C++ backend
void setMyMethodobjInCPP(arma::vec t_resid) {
  if (ptr_gMyMethodObj) delete ptr_gMyMethodObj;
  ptr_gMyMethodObj = new MyMethod::MyMethodClass(t_resid);
}

// 4. Method dispatch inside mainMarkerInCPP() function
if (t_method == "MyMethod") {
  ptr_gMyMethodObj->getAnyResult(GVec);
}
```

---

## 2. Examples for Each Function

### 2.1 `GRAB.MyMethod`

**Purpose**: User-facing documentation.

**Example**:

```r
#' MyMethod performs association tests for [trait type]
#'
#' @examples
#' obj <- GRAB.NullModel(pheno ~ AGE, method = "MyMethod", ...)
#' GRAB.Marker(obj, GenoFile, OutputFile)
#'
GRAB.MyMethod <- function() {
  .message("?GRAB.MyMethod for instructions")
}
```

---

### 2.2 `checkControl.NullModel.MyMethod`

**Purpose**: Validate inputs and set default control parameters.

**Example**:

```r
checkControl.NullModel.MyMethod <- function(traitType, GenoFile, SparseGRMFile, control) {
  # Validate trait type
  if (!traitType %in% c("quantitative", "binary")) {
    stop("MyMethod supports 'quantitative' or 'binary' traits only.")
  }
  
  # Set defaults
  default.control <- list(
    max_iter = 100,
    tolerance = 1e-6
  )
  control <- updateControl(control, default.control)
  
  # Validate parameters
  if (control$max_iter < 1) {
    stop("control$max_iter must be >= 1")
  }
  
  return(list(control = control, optionGRM = NULL))
}
```

---

### 2.3 `fitNullModel.MyMethod`

**Purpose**: Fit null model and prepare objects for testing.

**Example**:

```r
fitNullModel.MyMethod <- function(response, designMat, subjData, control) {
  # Fit model
  lm_fit <- lm(response ~ designMat)

  # Prepare other objects for step 2
  # ......
  
  # Return S3 object
  re <- list(
    N = length(subjData),
    subjData = subjData,
    resid = lm_fit$residuals
  )
  
  class(re) <- "MyMethod_NULL_Model"
  return(re)
}
```

---

### 2.4 `checkControl.Marker.MyMethod`

**Purpose**: Validate marker-level control parameters.

**Example**:

```r
checkControl.Marker.MyMethod <- function(control) {
  # Set defaults
  default.control <- list(
    use_firth = FALSE
  )
  control <- updateControl(control, default.control)
  
  # Validate
  if (!is.logical(control$use_firth)) {
    stop("control$use_firth must be TRUE or FALSE")
  }
  
  return(control)
}
```

---

### 2.5 `setMarker.MyMethod`

**Purpose**: Initialize C++ backend with null model objects.

**Example**:

```r
setMarker.MyMethod <- function(objNull, control) {
  setMyMethodobjInCPP(
    t_resid = objNull$residuals,
    ...
  )
  invisible(NULL)
}
```

---

### 2.6 `mainMarker.MyMethod`

**Purpose**: Call C++ backend and format output.

**Example**:

```r
mainMarker.MyMethod <- function(genoType, genoIndex) {
  # Call C++
  OutList <- mainMarkerInCPP(
    t_method = "MyMethod",
    t_genoType = genoType,
    t_genoIndex = genoIndex
  )
  
  # Format output
  data.frame(
    Marker = OutList$markerVec,
    Pvalue = OutList$pvalVec,
    ...
  )
}
```

---

### 2.7 `MyMethodClass`

**Purpose**: Store common data in the class and calculate result of one SNP.

**C++ Example**:

`src/MyMethod.h`:

```cpp
#include <RcppArmadillo.h>

namespace MyMethod {
  class MyMethodClass {
  private:
    arma::vec m_resid; // Residuals vector

  public:
    MyMethodClass(const arma::vec& t_resid);
    double getAnyResult(const arma::vec& GVec);
  };
}
```

`src/MyMethod.cpp`:

```cpp
#include "MyMethod.h"

namespace MyMethod {

  // Store common data in the class
  MyMethodClass::MyMethodClass(const arma::vec& t_resid) {
    m_resid = t_resid;
  }

  // Get result of one marker
  double MyMethodClass::getAnyResult(const arma::vec& GVec) {
    return arma::dot(GVec, m_resid);
  }
}
```
