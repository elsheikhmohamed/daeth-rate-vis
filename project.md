---
follows: dataLit
id: litvis
---

# Data Visualization Project Summary

{(whoami|} Mohamed El Sheikh
mohamed.el-sheikh@city.ac.uk {|whoami)}

{(task|}

You should complete this datavis project summary document and submit it, along with any necessary supplementary files to **Moodle** by **Sunday 18th December, 5pm UK time**. Submissions will be awarded up to **80 marks** towards your coursework assessment total.

You are also encouraged to regularly commit and push changes to your datavis project throughout the term as you develop your project.

{|task)}

{(questions|}

My main aim from this project is to understand the Death rate of the UK and the effect of Covid-19 on it. The data i will be using are from 2006-2020. This will allow me to understand how does death rate decrease/increase and what are the main factors that led to this reduction/increasing in the UK. I have collected different date sets from different sources (See references) these data will help me answer my research questions:

1. How does death rate change through different regions in the UK?
   i. where is it the lowest?
   ii. what year does a noticeable change happen?
2. At the start of Covid (2020) were people dying instantly from Covid?  
   i. what was the main cause for death
   ii. did more males or females die
3. How did Covid change the death rate for the UK

{|questions)}

{(visualization|}

### Exploring UK Death data 2006 - 2020

**Visualisation 1 (V1)**: Overall death rate for the UK from 2006-2020

```elm {v}
totalDeathRate : Spec
totalDeathRate =
    let
        encDate =
            encoding
                << position X [ pName "Date", pTemporal ]

        encDeath =
            encoding
                << position Y [ pName "Deaths", pQuant ]

        specDeath =
            asSpec [ encDeath [], bar [] ]

        encTimes =
            encoding
                << position Y [ pName "Region", pTitle " " ,pQuant ]

        specTime =
            asSpec [ encTimes [], line [] ]

        res =
            resolve
                << resolution (reScale [ ( chY, reIndependent ) ])
    in
    toVegaLite
        [ width 600
        , ukDeath
        , encDate []
        , res []
        , layer [ specDeath, specTime ]
        ]
```

**Visualisation 2 (V2)**: Different region total UK death rate 2006-2020 (interactive). Drag and select specific time to view the change in death rate in different regions.

```elm {v interactive}
totalDeathRateInteractive : Spec
totalDeathRateInteractive =
    let
        ps =
            params
                << param "mySelection" [ paSelect seInterval [ seEncodings [ chX ] ] ]
    in
    toVegaLite
        [ width 540
        , ukDeath
        , ps []
        , encHighlight []
        , bar []
        ]
```

**Visualisation 3 (V3)**: Change of death rate throughout different regions of the UK

```elm {v}
deathVisUK : Spec
deathVisUK =
    let
        enc =
            encoding
                << position X [ pName "Date", pTemporal, pTitle "" ]
                << position Y [ pName "Deaths", pAggregate opSum, pTitle "" ]
                << color [ mName "Region", mLegend [] ]
                << row
                    [ fName "Region"
                    , fHeader [ hdTitleFontSize 0 ]
                    ]

        res =
            resolve
                << resolution (reScale [ ( chY, reIndependent ) ])
    in
    toVegaLite [ height 100, width 455, ukDeath, line [], enc [] ]
```

### Exploring UK's 2020 Death cause data

**Visualisation 4 (V4)**: The main death cause reported in 2020 in the UK for males and females

```elm {v}
sexDeath : Spec
sexDeath =
    let
        enc =
            encoding
                << position X [ pName "Sex", pTitle "" ]
                << position Y [ pName "% of deaths", pQuant]
                << color [ mName "Leading Cause", mTitle "Leading Cause" ]
    in
    toVegaLite [ width 70, ukDeath2020 [], bar [], enc [] ]
```

### Exploring UK's Total COVID-19 Death data

**Visualisation 5 (V5)**: The increase of death caused by COVID-19

```elm {v}
waterfallCovidDeath : Spec
waterfallCovidDeath =
    let
        data =
            dataFromColumns []
                << dataColumn "label" (strs [ "Current", "2022", "2021", "2020", "Begin" ])
                << dataColumn "amount" (nums [ 0, 22999806, 47328573, 11758505, 0 ])

        trans =
            transform
                << window [ ( [ wiAggregateOp opSum, wiField "amount" ], "sum" ) ] []
                << window [ ( [ wiOp woLead, wiField "label" ], "lead" ) ] []
                << calculateAs "datum.lead === null ? datum.label : datum.lead" "lead"
                << calculateAs "datum.label === 'Begin' ? 0 : datum.sum - datum.amount" "previous_sum"
                << calculateAs "datum.label === 'Begin' ? datum.sum : datum.amount" "amount"
                << calculateAs "(datum.label !== 'Current' && datum.label !== 'Current' && datum.amount > 0 ? '+' : '') + datum.amount" "text_amount"
                << calculateAs "(datum.sum + datum.previous_sum) / 2" "center"
                << calculateAs "datum.sum < datum.previous_sum ? datum.sum : ''" "sum_dec"
                << calculateAs "datum.sum > datum.previous_sum ? datum.sum : ''" "sum_inc"

        enc =
            encoding
                << position X [ pName "label", pOrdinal, pSort [], pTitle "" ]

        enc1 =
            encoding
                << position Y [ pName "previous_sum", pQuant, pTitle "Death of Covid-19" ]
                << position Y2 [ pName "sum" ]
                << color
                    [ mConditions
                        [ ( prTest (expr "datum.label === 'Current' || datum.label === 'Current'"), [ mStr "#f7e0b6" ] )
                        , ( prTest (expr "datum.sum < datum.previous_sum"), [ mStr "#f78a64" ] )
                        ]
                        [ mStr "#D7263D" ]
                    ]

        spec1 =
            asSpec [ enc1 [], bar [ maSize 45 ] ]

        enc2 =
            encoding
                << position X2 [ pName "lead" ]
                << position Y [ pName "sum", pQuant ]

        spec2 =
            asSpec
                [ enc2 []
                , rule
                    [ maColor "#404040"
                    , maOpacity 1
                    , maStrokeWidth 2
                    , maXOffset -22.5
                    , maX2Offset 22.5
                    ]
                ]

        enc3 =
            encoding
                << position Y [ pName "sum_inc", pQuant ]
                << text [ tName "sum_inc" ]

        spec3 =
            asSpec
                [ enc3 []
                , textMark
                    [ maDy -8
                    , maFontWeight fwBold
                    , maColor "#404040"
                    ]
                ]

        enc4 =
            encoding
                << position Y [ pName "sum_dec", pQuant ]
                << text [ tName "sum_dec" ]

        spec4 =
            asSpec
                [ enc4 []
                , textMark
                    [ maDy 8
                    , maBaseline vaTop
                    , maFontWeight fwBold
                    , maColor "#404040"
                    ]
                ]

        enc5 =
            encoding
                << position Y [ pName "center", pQuant ]
                << text [ tName "text_amount" ]
                << color
                    [ mCondition (prTest (expr "datum.label === 'Begin' || datum.label === 'Current'"))
                        [ mStr "#725a30" ]
                        [ mStr "white" ]
                    ]

        spec5 =
            asSpec [ enc5 [], textMark [ maBaseline vaMiddle, maFontWeight fwBold ] ]
    in
    toVegaLite
        [ width 400
        , height 300
        , data []
        , trans []
        , enc []
        , layer [ spec1, spec2, spec3, spec4, spec5 ]
        ]
```

**Visualisation 6 (V6)**: The increase of COVID-19 death from 2020 to 2022

```elm {v}
minimalDotCovid : Spec
minimalDotCovid =
    let

        encRects =
            encoding
                << position X [ pName "start", pTimeUnit year ]
                << position X2 [ pName "end", pTimeUnit year ]


        specRects =
            asSpec [  encRects [], rect [ maOpacity 0.5 ] ]

        encPopulation =
            encoding
                << position X [ pName "date", pTemporal, pAxis [ axTitle "2020 ------------------------------------------------2022" ,axValues (nums [ 0.5 ])]]
                << position Y [ pName "cumDeaths28DaysByDeathDate", pQuant, pAxis [] ]

        specLine =
            asSpec
                [ encPopulation []
                , line
                    [ maColor "red"
                    , maInterpolate miMonotone
                    , maPoint (pmMarker [ maColor "black" ])
                    ]
                ]
    in
    toVegaLite [ width 500, covidDeathUK2022 [], layer [ specRects, specLine ] ]
```

{|visualization)}

{(insights|}

#### Data Understanding

This project uses 4 main datasets, all relating to the Uk's death rate in different regions from 2006 to 2020. It also contains COVID-19 death data from 2020 to 2022 (current):

- UK Death rate. The dataset contains The death rate of different regions of the UK from the year 2006 until the year 2020.

- UK Death cause 2020 data. Different types of death caused by diseases, natural death, covid, cancer …etc. Males and females death and the cause of the death.

- Covid Death UK 2020 data. It contains, Month, Number of death , Reason. For the reason it contains whether the death was caused by COVID-19 or no

- Covid Death UK 2022. It contains number of death caused by COVID-19 from 2020 till 2022

In some situations, these datasets were altered outside to provide new datasets ideal for particular visualisations. These were occasionally supplemented with pertinent extra data, such as information on population and geographical administrative regions, or communes.

#### Exploring UK Death Data (V1-V2)

The first reasonable thing to do, given that we are interested in the UK's death rate, was to investigate the overall death rate in the UK. I began by talking about the year and the death caused for each year from because that seemed to be the most crucial information to comprehend regarding the overall death to monitor the behaviour of death rate.

Understanding patterns and trends in death data can provide insight into the health and well-being of a population and can inform public health policies and interventions. Some key insights that can be gleaned from death data (V1-V2). Looking at V1 we can see that the death rate was stable from late 2009 until early 2013 with no significant changes during this period. However, it seems like every beginning of a year there would be an increase of the death rate, this applies to every year. The increase does not go down in the following year, for instance, if in 2015 total death was 50000 then the next year it will be more than that. Interestingly, in 2020 we can see that there was a spike in the graph showing more death than normal. This was around March – April time, there was a significant change in the death rate. To no surprise this was due to COVID-19 hitting the UK.

To comprehend the variation in different regions in the UK in V2, I used an interactive layer graph. The graph was a useful tool for identifying potential patterns. If we select the whole graph, we can come to an interesting fact about South East. First of all, we can see a strange pattern in the death rate. We can notice that in the years, 2009,2015,2018 there was a higher number of death than other regions in the UK. The different colour of bar plots and tooltips' assistance also allowed us to determine that the peak year for death was 2020.

#### Different Regions Death Data (V3)

Firstly, There are several factors that can affect the death rate in a population, including:

- Demographics: Age is a significant factor in death rates. Older populations tend to have higher death rates than younger populations.

- Health behaviours: Lifestyle factors such as smoking, poor diet, and lack of exercise can increase the risk of death from preventable causes.

- Access to healthcare: Lack of access to quality healthcare can lead to higher death rates, particularly for those with chronic illnesses or those in need of emergency care.

- Environmental factors: Exposure to toxins or hazardous conditions in the environment can increase the risk of death.

- Socioeconomic status: People living in poverty or with low socio-economic status may have higher death rates due to a lack of access to resources and opportunities that can impact health.

- Infectious diseases: The spread of infectious diseases can lead to increased death rates, particularly in populations with low immunity or limited access to healthcare.

- Natural disasters: Severe weather events and other natural disasters can lead to increased death rates due to injuries and other health complications.

Therefore, we can see the noticeable change in death rate throughout different regions. We have 12 different regions in the UK, we can categorise them into two different categories. A: stable pattern, B: unstable pattern. For category A we have regions like North East, if we look at its graph we can see that the death rate we very stable throughout the years. Strangely, even during the beginning of Covid it was not as hight as other regions. I am assuming that North East either followed the lockdown restriction strictly or they have a really good health profile and they are more immune than people who live in North West for example.

Overall, in recent years, the death rate in the UK has generally been declining, due in part to improvements in healthcare, public health measures, and advances in medical technology. However, the COVID-19 pandemic has had a significant impact on the death rate in the UK, with a significant increase in the number of deaths due to the virus in 2020 and early 2021. The death rate in the UK was also affected by seasonal influenza outbreaks and other causes of mortality, such as cardiovascular disease and cancer. It is important to note that the death rate can vary significantly by region and demographic group, and can be influenced by a wide range of social, economic, and environmental factors.

#### Male and female death and the impact of COVID-19 2020-2022 (V4 V5)

Looking at visualisation (V4) we can see that more men have died than women in 2020. If we look closely at the graph, we can see death of males were more reported than females. Surprisingly, there were a lot of death because of ‘self-harm’. There are several factors that contribute to the fact that men generally have a higher mortality rate than women, however, men are more likely to engage in risky behaviours such as smoking, heavy alcohol consumption, and dangerous sports or activities. These behaviours can increase the risk of serious health problems and death. Also, Covid-19 at the beginning has caused the death of more men than women. There is some evidence to suggest that women may have a stronger immune system than men, at least in certain contexts. For example, studies have shown that women tend to have a stronger immune response to vaccines, and that they are less likely to develop certain infectious diseases. I think this kind of explains why more men have died of Covid-19 than women at the beginning.

Looking at (V5) we can see the signific change in death rate in the UK after covid-19 hitting the UK in serious waves. It had significant impact on global health and has resulted in many deaths worldwide not only in the UK. However, in the United Kingdom, COVID-19 has been a significant cause of morbidity and mortality. According to (V5) the UK government, as of December 2021, the total number of confirmed COVID-19 cases in the UK was over 4.3 million. These numbers have fluctuated over time as the pandemic has progressed and as various public health measures have been implemented. It's important to note that the actual number of COVID-19 cases and deaths may be higher than the reported numbers, as not all cases are diagnosed or reported. It's also important to remember that the COVID-19 pandemic has had a wide range of impacts beyond just the direct deaths from the disease, including significant disruption to daily life and the economy, as well as indirect effects on mental health and well-being.

{|insights)}

{(designJustification|}

#### Exploring UK Death Data (V1-V2)

### Color

Choosing the right colours for data visualization is important because colours can impact the effectiveness and interpretability of the visualization. Here are some reasons why choosing the right colours is important in data visualization. Colour can help encode information: By choosing colours that reflect the data being visualized, you can use colour to help convey information and make the visualization more meaningful. For example, I have used red to indicate the negative values of COVID-19’s death in my last graph. The color scheme allows us to clearly distinguish the most and least important data in the char, for example, in my V4 i have used different colors to differenate

Since some colours can impact accessibility, i have used color combinations that made it easily distinguishable for people with colorblindness, so it is important to choose colors that are easily distinguishable for all viewers.

It is also important to keep in mind that different cultures may associate different meanings with certain colours. For example, red may be associated with danger or warning in some cultures(Jo Wood L5), while it may be associated with good luck or happiness in others. Therefore, it is important to consider the audience and context when using colours in data visualization. In the past decade, the primary theoretical advances in the area of colour and psychological functioning have shared a common feature: They have sought to ground colour effects in biology, drawing on parallels between human and nonhuman responding to colour stimuli. The germ of these ideas has been present for quite some time and noted by several different scholars (Andrew J 98)

### Interactive

Interactive graphs are useful in data visualization because they allow the user to explore and interact with the data in a more dynamic way. This can be particularly useful when the data is complex or there are many data points, as it allows the user to focus on the specific data points or trends that are most relevant to them. This is why i have used an interactive graph for my V2 to allow users to draw focus on specific years if they wanted to. My graph uses << param "mySelection" which allows users to click and drag a specific place on the graph to show more features, for instance, for me it was colors for different regions.

It also allows the viewers to interact with the visualization and explore the data in more depth(Randy Krum p198). Interactive graphs typically allow the viewer to zoom in on specific areas of the visualization, filter the data, and highlight specific data points.

### Minimalism

Throughout, my visualisations I have followed one simple rule. Show less, tell more. I have followed a minimal approach to my visualisations. My approach emphasizes simplicity and clarity by using a minimal number of elements to convey the necessary information. This involved using simple shapes, colours, and layout to create a clean and uncluttered visual representation of the data. Minimal visualizations can be particularly effective in cases where the data is complex or there is a large amount of information to convey, as they help to focus the viewer's attention on the most important aspects of the data and reduce distractions. As shown in my previous graphs, I have used very simple graphs, yet, I showed a lot of information.

According to Ed Swires-Hennessy (p43) One key aspect of minimal design in data visualization is the use of simple and clean visual elements, such as line graphs or bar charts, to represent the data. These types of visualizations can be effective in communicating trends, patterns, and relationships in the data, and can help to focus the viewer's attention on the most important aspects of the data. Another important aspect of minimal design in data visualization is the use of clear and concise labels, titles, and captions to help the viewer understand the data. It may also involve using a limited colour palette to avoid distractions and help the viewer focus on the most important aspects of the data.

In my V6 I have followed Tufte's principle. Some of the principles that 
i followed are; Showing comparisons: to make it easy for the viewer to compare different data points by using appropriate visual encodings, such as position, length, angle, area, or color. Showing causality: Show less labels that satisfies what i wanted to show the viewers, for instance, in my V6 i have used only one label "2020 --- 2022" on the x-axis to show where the 'year' data start and where it reaches. In addition, i have used (specRects) enc to display connected lines to show the amount of death that has been reported throughout these 3 years (2020,2021,2022). I have also used Layers to draw a dot out where each line begins and ends to show the density of death per year. The further away the dots the more death occurred during the time. 


{|designJustification)}

{(references|}

**Andrew J. Elliot1 and Markus A. Maier**: (2014) Color Psychology: Effects of Perceiving Color on Psychological Functioning in Humans

**Jo Wood**: (2022) Data visualization Lectures

**Randy Krum** (2013) Cool Infographics: Effective Communication with Data Visualization and Design

**by Abha Belorkar (Author), Sharath Chandra Guntuku** (2020) Interactive Data Visualization with Python: Present your data as an effective and compelling story, 2nd Edition

**Ed Swires-Hennessy** (2014) Presenting Data: How to Communicate Your Message Effectively

**Tufte, E.** (2001) [The Visual Display of Quantitative Information](https://go.exlibris.link/HRpwwyBl), Graphics Press.

**Data Reference**: Gov.uk

{|references)}
