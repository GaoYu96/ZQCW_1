# demo-test

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

```

账号:ceshi1
密码：123456


首页左上角的端州区、鼎湖区tab页，点击后分别跳转相应路由otherLinks、dinghu，以上两页面均使用iframe内嵌外部系统登录页（端州区及鼎湖区各自的网格化信息管理平台）。
若点击高要，则放大地图左侧展示高要区的网格事件

点击首页右上角的后台管理按钮，则向后端发送请求，接口返回checkLogin标志。
 然后用 window.open(href, '_blank')方式打开新页面，href拼接登录系统时后端返回的token及该请求接口返回的checkLogin标志
 var href=`http://192.168.0.242:8084/zhaoqing/index_test?token=${mytoken}&checkLogin=${this.checkLogin}`
 (mainHome.vue 152-165 行)

首页左侧悬浮的红色箭头，鼠标移入可选择首页显示左右两边echarts图表的主题，分为人员类型和事件类型。

地图放大后页面右侧区域信息框显示的信息取决于 el-cascader 级联选择器下的值。


端州区下层级分为：街道>社区>网格
鼎湖区下层级分为：街道>网格
高要区下层级分为：街道>社区>网格 或 街道>网格 或 村>村网格



如果当前区域为鼎湖区的话，需进行特殊处理,因为在进入鼎湖区网格时this.curLevel的取值为社区，需要设置为网格才能获取正确的区域信息
  watch: {
    curGrid: {
      //市级 县区 街道 社区 网格
      handler(n, o) {
        this.$nextTick(() => {
          let path = [
            ...this.$refs["cascader"].getCheckedNodes()[0].pathLabels
          ];
          if (typeof n !== "string") {
            this.curLevel = this.levelList[n.length - 1];
            this.curArea = n.length == 1 ? null : path[n.length - 1];
          } else {
            this.curLevel = path[0];
            this.curArea = null;
          }
         if(path.length>=4&&path[1]==="鼎湖区"){
            this.getAreaInfo({
            areaName: this.curArea,
            level: "网格",
             area:path[1],
            field:this.myField
          });
          }
          else{
            this.getAreaInfo({
            areaName: this.curArea,
            level: this.curLevel,
            area:path[1],
            field:this.myField
          });
          }
          path.shift();
          this.$refs.map.setPath(path);
        });
        // this.curArea = path == 1 ? null : path[path.length - 1];
      }
    }
  },

县区、街道、社区、网格分别绑定点击事件，点击区域则在地图上打开信息窗口（this.map.openInfoWindow(infoWindow, infoPoint)），展示区域信息同时隐藏右侧区域信息块。关闭信息窗口时，显示右侧区域信息块

鼎湖区下的社区点击事件需要做特殊处理，因为鼎湖区没有社区层级，需要手动将获取区域信息接口的level参数改为网格

县区：(zqmap.vue 602-750行)
  ply.addEventListener("click", e => {
        let infoPoint = "";
        let infoWindow = "";
        let opts = "";
        let infoMarker = "";
        let infoContent = "";
        let isInfo = "";
        let infoPointArr = [];
        var field=sessionStorage.getItem('field')
        this.getMyInfo(item.name, "县区",item.name,field);
        clearTimeout(clickTimeId);
        clickTimeId = setTimeout(() => {
          switch (this.RDetail.areaName) {
            case "端州区":
              infoPoint = "112.48138,23.079366";
              break;
            case "鼎湖区": //
              infoPoint = "112.652705,23.18784";
              break;
            case "高要区":
              infoPoint = "112.459533,22.945247";
              break;
          }

          infoPointArr = infoPoint.split(",");
          infoPoint = new BMap.Point(infoPointArr[0], infoPointArr[1]);
          let oDiv = document.createElement("div");
             if(this.noAdmin){
                    infoContent=`无权限查看`
                  }
                  else{
                     infoContent = `<h2>${this.RDetail.areaName}</h2>
                  街道乡镇：
                  ${this.RDetail.belong}<br>
              网格数：
              ${this.RDetail.gridNum}<br>
              相关人数：
              <a style="color:red;cursor: pointer" class="clickable">
              ${this.RDetail.relations} </a>
              <h3>事件情况</h3>
              事件数量：
              
              ${this.RDetail.eventNum}<br>
              ${(this.RDetail.counts||[])
                .map(item => `${item.thingType}: ${item.count}<br>`)
                .join("")}
                    事件办结率：
              ${this.RDetail.eventRate}`;
                  }
         
          oDiv.innerHTML = infoContent;
          setTimeout(() => {
            this.$nextTick(() => {
              document
                .getElementsByClassName("clickable")[0]
                .addEventListener("click", () => {
                  this.emitPersonNumGetList(
                    this.RDetail.areaName,
                    this.RDetail.level,
                    true,
                   this.RDetail.areaName,
                    "a"
                  );
                });
             
            });
          }, 200);
          opts = {
            width: 200, // 信息窗口宽度
            height: 230 // 信息窗口高度
          };
          infoWindow = new BMap.InfoWindow(infoContent, opts); // 创建信息窗口对象
          this.map.openInfoWindow(infoWindow, infoPoint);
          setTimeout(() => {
            this.$store.commit("isOpen", infoWindow.isOpen());
          }, 150);
          infoWindow.addEventListener("clickclose", e => {
            //信息窗口关闭
            setTimeout(() => {
              this.$store.commit("isOpen", infoWindow.isOpen());
              // this.isInfoOpen(infoWindow.isOpen());
            }, 100);
          });
        }, 250);
      });


街道：(zqmap.vue 356-536行)
          streeItem.addEventListener("click", e => {
          
            var field=sessionStorage.getItem("field")
            this.getMyInfo(item.belongStreet, "街道",item.belongArea,field);
            clearTimeout(clickTimeId);
            clickTimeId = setTimeout(() => {
              this.streetData.forEach((item, inx) => {
                if (item.belongStreet === this.RDetail.areaName) {
                  console.log(this.RDetail,"this.RDetail");
                 
                  if(!this.noAdmin){}
                  let infoWinPoint = "";
                  let infoWinPointArr = [];
                  let centerArr = "";
                  let centerPoint = [];
                  infoWinPoint = item.center; //string
                  let num = item.center.indexOf("&"); //一个街道分成多块的情况
                  if (num > -1) {
                    centerArr = String(item.center.split("&"));
                    let centerPoint = centerArr.split(",");
                    infoWinPointArr = centerPoint;
                  } else {
                    infoWinPoint = item.center;
                    infoWinPointArr = infoWinPoint.split(",");
                  }
                  infoPoint = new BMap.Point(
                    infoWinPointArr[0],
                    infoWinPointArr[1]
                  );

                  let oDiv = document.createElement("div");

                  if(this.noAdmin){
                    infoContent=`无权限查看`
                  }
                  else{
                    infoContent = `<h2>${this.RDetail.areaName}</h2>
              网格数：
              ${this.RDetail.gridNum}<br>
              相关人数：
              <a style="color:red;cursor: pointer" class="clickable">
              ${this.RDetail.relations} </a>
              <h3>事件情况</h3>
              事件数量：
              
              ${this.RDetail.eventNum}<br>
              ${(this.RDetail.counts||[])
                .map(item => `${item.thingType}: ${item.count}<br>`)
                .join("")}
                    事件办结率：
              ${this.RDetail.eventRate}`;
                  }
                  
                  oDiv.innerHTML = infoContent;
                  setTimeout(() => {
                    this.$nextTick(() => {
                      document
                        .getElementsByClassName("clickable")[0]
                        .addEventListener("click", () => {
                          this.emitPersonNumGetList(
                            this.RDetail.areaName,
                            this.RDetail.level,
                            true,
                            item.belongArea,
                            "a"
                          );
                        });
                    });
                  }, 200);
                  opts = {
                    width: 200, // 信息窗口宽度
                    height: 200 // 信息窗口高度
                    // title: this.RDetail.areaName // 信息窗口标题
                  };
                  infoWindow = new BMap.InfoWindow(infoContent, opts); // 创建信息窗口对象
                  this.map.openInfoWindow(infoWindow, infoPoint);
                  setTimeout(() => {

                    console.log(infoWindow.isOpen())
                    this.$store.commit("isOpen", infoWindow.isOpen());
                  }, 150);
                  infoWindow.addEventListener("clickclose", e => {
                    setTimeout(() => {                       
                      this.$store.commit("isOpen", infoWindow.isOpen());

                    }, 150);
                  });
                }
              });
            }, 250);
          });



社区：(zqmap.vue 1182-1481行)                                            鼎湖区需要特殊处理
 gridItem.addEventListener("click", e => {
              clearTimeout(clickTimeId);
              let CinfoPoint = "";
              let infoWindow = "";
              let opts = "";
              let infoMarker = "";
              let infoContent = "";
              let isInfo = "";
              var field=sessionStorage.getItem("field")
              if (this.path[0] == "鼎湖区" && this.curLevel == 1) {
                this.getMyInfo(item.belongCommunity, "网格",item.belongArea,field);                
                clickTimeId = setTimeout(() => {
                  this.communityData.forEach((item, inx) => {
                    if (item.belongCommunity == this.RDetail.areaName) {
                      let infoWinPoint = "";
                      let infoWinPointArr = [];
                      let centerArr = "";
                      let num = item.center.indexOf("&"); //一个社区分成多块的情况
                      if (item.belongArea == "鼎湖区") {
                        setTimeout(() => {
                          if (num > -1) {
                            centerArr = String(item.center.split("&"));
                            infoWinPointArr = centerArr.split(",");
                          } else {
                            infoWinPoint = item.center;
                            infoWinPointArr = infoWinPoint.split(",");
                          }
                          CinfoPoint = new BMap.Point(
                            infoWinPointArr[0],
                            infoWinPointArr[1]
                          );
                          let CoDiv = document.createElement("div");
                           if(this.noAdmin){
                    infoContent=`无权限查看`
                  }
                  else{
                     infoContent = `<h2>${this.RDetail.areaName}</h2>
                          所属单位：
                      ${this.RDetail.belongUnit}<br>
                      相关人数：
                      <a style="color:red;cursor: pointer" class="Cclickable">
                      ${this.RDetail.relations} </a><br>
                      <h3>事件情况</h3>
                      事件数量：
                     ${this.RDetail.eventNum}<br>
                      ${(this.RDetail.counts||[])
                        .map(item => `${item.thingType}: ${item.count}<br>`)
                        .join("")}
                      <h3>${this.MydetailLabel.griders}:</h3>             
               ${this.RDetail.griders
                 .map(
                   item =>
                     `${item.userName} ${item.phone ? +item.phone : ""}<br>`
                 )
                 .join("")}
                <h3>${
                  this.MydetailLabel.coordinator
                    ? this.MydetailLabel.coordinator + ":"
                    : ""
                }</h3>
              ${this.RDetail.coordinator
                .map(
                  item =>
                    `${item.userName} ${item.phone ? +item.phone : ""}<br>`
                )
                .join("")}`;
                  }
                         
                          CoDiv.innerHTML = infoContent;
                          setTimeout(() => {
                            this.$nextTick(() => {
                              document
                                .getElementsByClassName("Cclickable")[0]
                                .addEventListener("click", () => {
                                  this.emitPersonNumGetList(
                                    this.RDetail.areaName,
                                    this.RDetail.level,
                                    true,
                                    item.belongArea,
                            "a"
                                  );
                                });
                            });
                          }, 400);
                          opts = {
                            width: 200,
                            height: 270
                          };
                          infoWindow = new BMap.InfoWindow(infoContent, opts); // 创建信息窗口对象
                          this.map.openInfoWindow(infoWindow, CinfoPoint);
                          setTimeout(() => {
                            this.$store.commit("isOpen", infoWindow.isOpen());
                          }, 100);
                          infoWindow.addEventListener("clickclose", e => {
                            setTimeout(() => {
                              this.$store.commit("isOpen", infoWindow.isOpen());
                            }, 100);
                          });
                        }, 400);
                      }
                    }
                  });
                }, 400);
              } else {
                this.getMyInfo(item.belongCommunity, "社区",item.belongArea,field);     
                clickTimeId = setTimeout(() => {
                  this.communityData.forEach((item, inx) => {
                    let infoWinPoint = "";
                    let infoWinPointArr = [];
                    let centerArr = "";
                    let num = item.center.indexOf("&");
                   
                    if (item.belongCommunity == this.RDetail.areaName) {
                     
                      if (num > -1) {
                        centerArr = String(item.center.split("&"));
                        infoWinPointArr = centerArr.split(",");
                      } else {
                        infoWinPoint = item.center;
                        infoWinPointArr = infoWinPoint.split(",");
                        
                      }
                     
                      CinfoPoint = new BMap.Point(
                        infoWinPointArr[0],
                        infoWinPointArr[1]
                      );
                      let CoDiv = document.createElement("div");
                      infoContent = `<h2>${this.RDetail.areaName}</h2>
                      所属单位：
                      ${this.RDetail.belongUnit}<br>
              网格数：
              ${this.RDetail.gridNum}<br>
              相关人数：
              <a style="color:red;cursor: pointer" class="Cclickable">
              ${this.RDetail.relations} </a><br>
              <h3>事件情况</h3>
              事件数量：            
              ${this.RDetail.eventNum}<br>
              ${(this.RDetail.counts||[])
                .map(item => `${item.thingType}: ${item.count}<br>`)
                .join("")}
              事件办结率：
              ${this.RDetail.eventRate}`;
                      CoDiv.innerHTML = infoContent;
                      setTimeout(() => {
                        this.$nextTick(() => {
                          document
                            .getElementsByClassName("Cclickable")[0]
                            .addEventListener("click", () => {
                              this.emitPersonNumGetList(
                                this.RDetail.areaName,
                                this.RDetail.level,
                                true,
                                item.belongArea,
                           "a"
                              );
                            });
                        });
                      }, 250);
                      opts = {
                        width: 200,
                        height: 200
                      };
                      // console.log(infoContent,opts,"infoContent");
                      infoWindow = new BMap.InfoWindow(infoContent, opts); // 创建信息窗口对象
                      
                      this.map.openInfoWindow(infoWindow, CinfoPoint);
                      setTimeout(() => {
                        this.$store.commit("isOpen", infoWindow.isOpen());
                      }, 250);
                      infoWindow.addEventListener("clickclose", e => {
                        setTimeout(() => {
                          // this.isInfoOpen(infoWindow.isOpen());
                          this.$store.commit("isOpen", infoWindow.isOpen());
                        }, 250);
                      });
                    }
                  });
                }, 250);
              }
            });


网格：(zqmap.vue 825-1030行) 
  points.addEventListener("click", e => {
            let GinfoPoint = "";
            let infoWindow = "";
            let opts = "";
            let infoMarker = "";
            let infoContent = "";
            let isInfo = "";
            var infoWinPoint;
             let centerArr = "";
            var field=sessionStorage.getItem("field")
            this.getMyInfo(item.belongGrid, "网格",item.belongArea,field);    
            clearTimeout(clickTimeId);
            clickTimeId = setTimeout(() => {
              this.gridData.forEach((item, inx) => {
                let infoWinPointArr = [];
             
                if (item.belongGrid === this.RDetail.areaName) {
            
                  infoWinPoint = JSON.stringify(item.center).replace(
                    /\[|]/g,
                    ""
                  );
                  let infoWinPoint1 = infoWinPoint
                    .replace('"', "")
                    .replace('"', "")
                    .replace('"', "")
                    .replace('"', "");
                  infoWinPointArr = infoWinPoint1.split(",");
                  GinfoPoint = new BMap.Point(
                    infoWinPointArr[0],
                    infoWinPointArr[1]
                  );
                  console.log(GinfoPoint);
                  let GoDiv = document.createElement("div");
                    if(this.noAdmin){
                    infoContent=`无权限查看`
                  }
                  else{
                     infoContent = `<h2>${this.RDetail.areaName}</h2>
              所属单位：
              ${this.RDetail.belongUnit}<br>
              相关人数：
              <a style="color:red;cursor: pointer" class="Gclickable">
              ${this.RDetail.relations} </a>
              <h3>事件情况</h3>
              事件数量：
              
              ${this.RDetail.eventNum}<br>
             ${(this.RDetail.counts||[])
               .map(item => `${item.thingType}: ${item.count}<br>`)
               .join("")}
              事件办结率：
              ${this.RDetail.eventRate}
               <h3>${this.MydetailLabel.griders}:</h3>             
               ${this.RDetail.griders
                 .map(
                   item =>
                     `${item.userName} ${item.phone ? +item.phone : ""}<br>`
                 )
                 .join("")}
                <h3>${
                  this.MydetailLabel.coordinator
                    ? this.MydetailLabel.coordinator + ":"
                    : ""
                }</h3>
              ${this.RDetail.coordinator
                .map(
                  item =>
                    `${item.userName} ${item.phone ? +item.phone : ""}<br>`
                )
                .join("")}`;
                  }
                 
                  GoDiv.innerHTML = infoContent;
                  setTimeout(() => {
                    this.$nextTick(() => {
                      document
                        .getElementsByClassName("Gclickable")[0]
                        .addEventListener("click", () => {
                          this.emitPersonNumGetList(
                            this.RDetail.areaName,
                            this.RDetail.level,
                            true,
                            item.belongArea,
                           "a"
                          );
                        });
                    });
                  }, 200);
                  console.log( document.getElementsByClassName("Gclickable")[0],"hhh");
                  let optHeight;
                  if (this.RDetail.coordinator.length > 2) {
                    optHeight = 320;
                  } else {
                    optHeight = 280;
                  }
                  opts = {
                    width: 200, // 信息窗口宽度
                    height: optHeight
                    // 信息窗口高度
                  };
                  infoWindow = new BMap.InfoWindow(infoContent, opts); // 创建信息窗口对象
                  this.map.openInfoWindow(infoWindow, GinfoPoint);
                  setTimeout(() => {
                    //开启信息窗口
                    this.$store.commit("isOpen", infoWindow.isOpen());
                  }, 200);
                }
              });
              infoWindow.addEventListener("clickclose", e => {
                setTimeout(() => {
                  this.$store.commit("isOpen", infoWindow.isOpen());
                }, 150);
              });
            }, 250);
          });





vue.config.js设置代理
  /* 使用代理 */
    proxy: {
      "/proxyApi": {
        /* 目标代理服务器地址 */
        // target: "http://119.125.233.71:8083",
        target: "http://192.168.0.242:8083",
        /* 允许跨域 */
        changeOrigin: true,
        pathRewrite: {
          "^/proxyApi": ""
        }
      },
    }

数据接口统一存放在router文件夹下的api.js

export const api =
  process.env.NODE_ENV === "development"
    ? "http://192.168.0.242:8083"
    : "http://192.168.0.242:8083";
    // ? "http://119.125.233.71:8083"
    // : "http://119.125.233.71:8083";


公司服务器环境：http://192.168.0.242:8083
肇庆云服务器：http://119.125.233.71:8083 （端口目前已关闭，无法访问）


获取地图瓦片图的ip地址
公司服务器环境：http://192.168.0.242:8095/tiles
肇庆云服务器：http://119.125.233.71:8095/tiles （端口目前已关闭，无法访问）

