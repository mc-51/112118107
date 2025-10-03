import pandas as pd

tasks_data = [
    {"ID": 1, "說明": "研擬計畫", "需時(天)": 1, "前置任務": []},
    {"ID": 2, "說明": "任務分配", "需時(天)": 4, "前置任務": [1]},
    {"ID": 3, "說明": "取得硬體", "需時(天)": 17, "前置任務": [1]},
    {"ID": 4, "說明": "程式開發", "需時(天)": 70, "前置任務": [2]},
    {"ID": 5, "說明": "安裝硬體", "需時(天)": 10, "前置任務": [3]},
    {"ID": 6, "說明": "程式測試", "需時(天)": 30, "前置任務": [4]},
    {"ID": 7, "說明": "撰寫使用手冊", "需時(天)": 25, "前置任務": [5]},
    {"ID": 8, "說明": "轉換檔案", "需時(天)": 20, "前置任務": [5]},
    {"ID": 9, "說明": "系統測試", "需時(天)": 25, "前置任務": [6]},
    {"ID": 10, "說明": "使用者訓練", "需時(天)": 20, "前置任務": [7, 8]},
    {"ID": 11, "說明": "使用者測試", "需時(天)": 25, "前置任務": [9, 10]}
]

df = pd.DataFrame(tasks_data)

# 初始化ES, EF, LS, LF, Slack
df["ES"] = 0
df["EF"] = 0
df["LS"] = float("inf")
df["LF"] = float("inf")
df["Slack"] = float("inf")

# 前向計算 (Early Start, Early Finish)
for i in range(len(df)):
    task_id = df.loc[i, "ID"]
    duration = df.loc[i, "需時(天)"]
    predecessors = df.loc[i, "前置任務"]

    if not predecessors:
        df.loc[i, "ES"] = 0
    else:
        max_ef = 0
        for pred_id in predecessors:
            max_ef = max(max_ef, df[df["ID"] == pred_id]["EF"].iloc[0])
        df.loc[i, "ES"] = max_ef
    df.loc[i, "EF"] = df.loc[i, "ES"] + duration

project_duration = df["EF"].max()

# 後向計算 (Late Start, Late Finish)
for i in range(len(df) - 1, -1, -1):
    task_id = df.loc[i, "ID"]
    duration = df.loc[i, "需時(天)"]

    successors = df[df["前置任務"].apply(lambda x: task_id in x)]

    if successors.empty:
        df.loc[i, "LF"] = project_duration
    else:
        min_ls = float("inf")
        for _, succ_row in successors.iterrows():
            min_ls = min(min_ls, succ_row["LS"])
        df.loc[i, "LF"] = min_ls
    df.loc[i, "LS"] = df.loc[i, "LF"] - duration

# 計算Slack
df["Slack"] = df["LF"] - df["EF"]

# 找出關鍵路徑
critical_path_tasks = df[df["Slack"] == 0]["ID"].tolist()

print("任務分析結果：")
print(df.to_string())
print(f"\n專案總工期： {project_duration} 天")
print(f"關鍵路徑上的任務ID： {critical_path_tasks}")

# 將結果保存到CSV文件
df.to_csv("task_analysis_results.csv", index=False, encoding="utf-8-sig")

# 準備甘特圖數據
gantt_data = []
for index, row in df.iterrows():
    gantt_data.append({
        "Task": f"任務{row["ID"]}: {row["說明"]}",
        "Start": row["ES"],
        "End": row["EF"],
        "Duration": row["需時(天)"]
    })

gantt_df = pd.DataFrame(gantt_data)
gantt_df.to_csv("gantt_chart_data.csv", index=False, encoding="utf-8-sig")

import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

# 讀取甘特圖數據
df_gantt = pd.read_csv("gantt_chart_data.csv")

# 將開始和結束時間轉換為日期格式，假設從第一天開始
df_gantt["Start_Date"] = pd.to_datetime(df_gantt["Start"], unit=\'D\', origin=\'2025-01-01\')
df_gantt["End_Date"] = pd.to_datetime(df_gantt["End"], unit=\'D\', origin=\'2025-01-01\')

# 繪製甘特圖
fig, ax = plt.subplots(figsize=(12, 8))

# 繪製每個任務的條形圖
for i, row in df_gantt.iterrows():
    ax.barh(row["Task"], row["Duration"], left=row["Start_Date"], height=0.8)

# 設置X軸為日期格式
ax.xaxis.set_major_locator(mdates.DayLocator(interval=10)) # 每10天一個主要刻度
ax.xaxis.set_major_formatter(mdates.DateFormatter(\'%Y-%m-%d\'))
plt.xticks(rotation=45, ha=\'right\')

# 設置Y軸標籤
ax.set_xlabel("日期")
ax.set_ylabel("任務")
ax.set_title("專案甘特圖")
ax.invert_yaxis() # 讓第一個任務在頂部顯示

plt.tight_layout()
plt.savefig("gantt_chart.png")
print("甘特圖已保存為 gantt_chart.png")
graph TD
    A[1: 研擬計畫]
    B[2: 任務分配]
    C[3: 取得硬體]
    D[4: 程式開發]
    E[5: 安裝硬體]
    F[6: 程式測試]
    G[7: 撰寫使用手冊]
    H[8: 轉換檔案]
    I[9: 系統測試]
    J[10: 使用者訓練]
    K[11: 使用者測試]

    A --> B
    A --> C
    B --> D
    C --> E
    D --> F
    E --> G
    E --> H
    F --> I
    G --> J
    H --> J
    I --> K
    J --> K

    style A fill:#F9F,stroke:#333,stroke-width:2px
    style B fill:#F9F,stroke:#333,stroke-width:2px
    style D fill:#F9F,stroke:#333,stroke-width:2px
    style F fill:#F9F,stroke:#333,stroke-width:2px
    style I fill:#F9F,stroke:#333,stroke-width:2px
    style K fill:#F9F,stroke:#333,stroke-width:2px
