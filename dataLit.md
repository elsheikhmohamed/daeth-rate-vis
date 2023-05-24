---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

Datasets declaration

```elm {v l=hidden}
ukDeath =
    dataFromUrl  "https://raw.githubusercontent.com/elsheikhmohamed/data/main/uk_deaths.csv" []

ukDeath2020 =
      dataFromUrl "https://raw.githubusercontent.com/elsheikhmohamed/data/main/death2020uk.csv"

covidDeathUK2020 =
      dataFromUrl "https://raw.githubusercontent.com/elsheikhmohamed/data/main/covidDeathUK2020.csv"

img1 =
      dataFromUrl "https://github.com/elsheikhmohamed/data/blob/main/graph1.png"

covidDeathUK2022 =
    dataFromUrl "https://raw.githubusercontent.com/elsheikhmohamed/data/main/CovidDeath2022UK.csv"


```

```elm {v l=hidden}
deathColor =
    categoricalDomainMap
        [ ( "COVID-19", "rgb(143, 174, 34)" )
        , ( "Congenital malformations deformations", "rgb(0, 3, 91" )
        , ( "Criminal damage and arson", "rgb(141,106,184)" )
        , ( "Intentional self-harm", "rgb(255,0,0)" )
        , ( "Accidental poisoning", "rgb(255,255,0)" )
        , ( "Ischaemic heart diseases", "rgb(139,0,0)" )
        , ( "Dementia and Alzheimer", "rgb(138,43,226)" )
        , ( "Breast cancer", "rgb(255,192,203)" )
        ]

encHighlight =
    encoding
        << position X [ pName "Date", pTemporal, pAxis [ axTitle "", axGrid False ] ]
        << position Y [ pName "Deaths", pQuant, pAxis [ axGrid False ] ]
        << color
            [ mCondition (prParamEmpty "mySelection")
                [ mName "Region"]
                [ mStr "black" ]
            ]
        << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1 ]
                [ mNum 0.1 ]
            ]

```
