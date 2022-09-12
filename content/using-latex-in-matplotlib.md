---
title: 在matplotlib中使用latex渲染
date: 2019-09-12 11:35:20
taxonomies:
    tags: 
        - matplotlib
        - latex
        - python
---

## 代码

在代码中加入如下的代码就可以在matplotlib中使用latex渲染文字

```python
import matplotlib
matplotlib.use("pgf")
pgf_with_custom_preamble = {
    # "font.size": 40,
    "pgf.rcfonts": False,
    "text.usetex": True,
    "pgf.preamble": [
        # math setup:
        r"\usepackage{unicode-math}",
        r"\usepackage[utf8]{inputenc}",
        r"\usepackage{xeCJK}",
        r"\setmainfont{Times New Roman} % Set main font for ASCII",
        r"\setsansfont{Times New Roman}",
        r"\setmonofont{Times New Roman}"
        r"\setCJKmainfont{SimSun}",
        r"\setCJKsansfont{SimSun}",
        r"\setCJKmonofont{SimSun}"
    ],
    "pdf.fonttype": 42,
}
matplotlib.rcParams.update(pgf_with_custom_preamble)
```

完整的代码如下：

```python
import matplotlib
matplotlib.use("pgf")
pgf_with_custom_preamble = {
    # "font.size": 40,
    "pgf.rcfonts": False,
    "text.usetex": True,
    "pgf.preamble": [
        # math setup:
        r"\usepackage{unicode-math}",
        r"\usepackage[utf8]{inputenc}",
        r"\usepackage{xeCJK}",
        r"\setmainfont{Times New Roman} % Set main font for ASCII",
        r"\setsansfont{Times New Roman}",
        r"\setmonofont{Times New Roman}"
        r"\setCJKmainfont{SimSun}",
        r"\setCJKsansfont{SimSun}",
        r"\setCJKmonofont{SimSun}"
    ],
    "pdf.fonttype": 42,
}
matplotlib.rcParams.update(pgf_with_custom_preamble)

import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns

sns.set()
df = pd.read_hdf('./label.h5')

## 性别分布
gender = df['gender']

gender = gender.fillna('unkonwn')
counts = pd.value_counts(gender)
plt.figure(figsize=(10, 8))
counts.plot.pie(legend=True, autopct=lambda p: '{:.2f}%({:.0f})'.format(
    p, (p/100)*counts.sum()), title="Gender distribution")
plt.savefig('./figures/gender.png')
plt.savefig('./figures/gender')

## 年龄分布

age = df['age']
plt.clf()
age = age.dropna()
sns.distplot(age)
plt.title('kde plot')
plt.savefig('./figures/age.png')
plt.savefig('./figures/age')

## 诊断分布

tmp = df.iloc[:, 2:]
plt.clf()
counts = tmp.sum() / tmp.shape[0]
counts.plot(kind='bar', title='Percentage')
plt.tight_layout()
plt.savefig('./figures/diagnose.png')

plt.savefig('./figures/diagnose')

```

结果如图：

![gender](gender.svg)

![age](age.svg)

![dianose](diagnose.svg)
