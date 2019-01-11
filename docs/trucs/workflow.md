- 了解3.6和3.7版本的不同，分块总结，比如中文翻译部分，其他特性等等

- 3.6是内部版本，3.7是外部版本。先往3.7版本加入3.6版本的特性，然后再打包（open build service）成openSUSE的distribution，最后再进行内部的build

---

### fitstor:

- osd:
    - osd status
    
- 读取失败：
    - iSCSI
    - NFS
    - 对象网关
    - 系统配置

- **警告** 页面

- 系统配置：
    - 日志

### openATTIC:

API记录器
语言切换

- osd:
    - crush 权重

### 不同：

**翻译有些不同**：
    - 配置OSD标示
    
    
---

enhanced:
1. lacking some testing codes 
2. /backend/rest/restapi.py:  user_permission
3. no rest/auth.py (the next point)
4. self-defined oa_auth module, but not using rest_auth directly
5. rest_session to TimeoutRequestsSession

---

enhanced features:

#### /backend (features)
- **user_permissions**: 
  - /backend/rest/restapi.py
  - /backend/oa_settings/views.py
  - /backend/ceph_radosgw/views.py
  - /backend/ceph_nfs/views (ganesha_export_viewset.py/ganesha_mgr_view.py)
  - /backend/ceph_iscsi/iscsi_{interface_viewset, mgr_view, target_viewset}.py
  - /backend/ceph/restapi.py

- **logging view**
  - /backend/oa_logging/urls.py
  - /backend/oa_logging/views.py
  - /webui/app/component/ceph-log

- **iops info**
  - /backend/ceph_iscsi/lrbd_conf.py
  - /webui/app/component/ceph-iscsi-form/
  - /webui/app/component/ceph-iscsi-form-image-settings-modal
  - /webui/app/component/shared/ceph-iscsi-image-optional-settings.value.js
  
- crush tree:
  - /backend/ceph/models.py

- **'alerts' component**:
  - /backend/alerts
  - /etc/addon/logging/views.py
  - /webui/app/components/ceph-alerts

#### /etc/addon


#### /webui (directory)

- protractor.conf.js: task_queue_directive & task_queue_deletion

- app
  - components
    - ceph-alerts: alert
    - ceph-iscsi:
      - ceph-iscsi-detail: image
      - **ceph-iscsi-form**-image-settings-modal: image, iops
      - ceph-iscsi-form: iops
    - ceph-log
    - ceph-osd
      - **ceph-osd-detail**
      - ceph-osd-list
      - ceph-osd.module.js
      - ceph-osd.route.js
- backup
- dist
- e2e
- locale
- node_modules

#### ???

翻译到底怎么进行？ --> advice: 自动更换系统
