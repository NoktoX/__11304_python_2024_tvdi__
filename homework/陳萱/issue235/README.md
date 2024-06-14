# [HW_issue216 連結](./index.py)

## index.py
```python
import tkinter as tk
from tkinter import ttk
from ttkthemes import ThemedTk
from tools import Youbike_Data

class Window(ThemedTk):
    def __init__(self, theme:str|None, **kwargs):
        super().__init__(**kwargs)


        self = tk.Tk()
        self.title("YouBike2.0 臺北市公共自行車即時資訊")
        self.geometry('1000x200')
        self.resizable(False, False)
        treeview_frame = ttk.Frame(self)
        treeview_frame.pack()

        # define columns
        treeview = ttk.Treeview(self,columns=('sna', 'sarea', 'mday', 'ar', 'act', 'updateTime', 'total', 'rent_bikes', 'lat', 'lng', 'retuen_bikes'),show='headings')

        
        # define headings
        treeview.heading('sna', text='站名')
        treeview.heading('sarea', text='站點區域')
        treeview.heading('mday', text='更新時間')
        treeview.heading('ar', text='地點')
        treeview.heading('act', text='是否可借')
        treeview.heading('updateTime', text='即時更新時間')
        treeview.heading('total', text='總車位數')
        treeview.heading('rent_bikes', text='可借車輛數')
        treeview.heading('retuen_bikes', text='可還空位數')
        treeview.heading('lat', text='緯度')
        treeview.heading('lng', text='經度')
        
        # 垂直和水平捲軸
        verticalsb = ttk.Scrollbar(treeview_frame, orient="vertical", command=treeview.yview)
        verticalsb.pack(side='right', fill='y')
        horizontalsb = ttk.Scrollbar(treeview_frame, orient="horizontal", command=treeview.xview)
        horizontalsb.pack(side='bottom', fill='x')

        # 設定Treeview的xscroll和yscroll
        treeview.configure(xscroll=horizontalsb.set, yscroll=verticalsb.set)
        treeview.pack()

        # 將資歷寫入到 Treeview
        try:
            ubike:list[dict] = ubike_tools.load_data()
        except Exception as error:
            print(error)
        else:
            for data in ubike:
                sna_value = data['sna'].split("_")[1]
                if data['act'] == '1':
                    act_value = "營業中" 
                else:
                    act_value = "維護中"
                treeview.insert('', 'end', values=(sna_value, data['sarea'], data['mday'], data['ar'], act_value, data['updateTime'], data['total'], data['rent_bikes'], data['lat'], data['lng'], data['retuen_bikes']))


def main():
    window = Window(theme='arc')
    window.mainloop()

if __name__ == '__main__':
    main()
```

## tools.py

```python
import requests
from requests import Response
from pydantic import BaseModel, RootModel, Field

def __download_json():
    url = "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"

    try:
        res:Response = requests.get(url)
    except Exception:
        raise ("連線失敗")
    else:
        all_data:dict[any] = res.json()
        return all_data

class Info(BaseModel):
    sna:str
    sarea:str
    mday:str
    ar:str
    act:str
    updateTime:str
    total:int
    rent_bikes:int = Field(alias="available_rent_bikes")
    lat:float = Field(alias="latitude")
    lng:float = Field(alias="longitude")
    retuen_bikes:int = Field(alias="available_return_bikes")

class Youbike_Data(RootModel):
    root:list[Info]

def load_data() -> list[dict]:
    all_data:dict[any] = __download_json()
    youbike_data:Youbike_Data = Youbike_Data.model_validate(all_data)
    data = youbike_data.model_dump()
    return data
```