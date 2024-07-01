# 坐标与海拔信息

>海拔信息（Elevation）指的是某个点相对于海平面的高度。在地理和导航应用中，了解某个地点的海拔高度可以提供关于该地点在垂直方向上的位置的信息，这对很多应用场景都非常重要，地形分析：了解地形的起伏变化，比如山脉、山谷、平原等。

已知经纬度信息情况下， 得到海拔信息
!!! tip 通过Open Elevation API获取到的海拔高度表示的是地面相对于海平面的高度


```python
# 获取海拔数据
def get_elevation(lat, lon):
    url = f'https://api.open-elevation.com/api/v1/lookup?locations={lat},{lon}'
    response = requests.get(url)
    if response.status_code == 200:
        results = response.json()['results']
        if results:
            print(results[0]['elevation'])
            return results[0]['elevation']
    return None

df_selected['Elevation'] = df_selected.apply(lambda row: get_elevation(row['Latitude'], row['Longitude']), axis=1)

# 保存提取的数据到新文件
output_file_path = './selected_recid_coordinates.csv'
df_selected.to_csv(output_file_path, index=False)
```