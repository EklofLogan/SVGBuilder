<script>
    import { onMount } from 'svelte';
    let app;
    let state = {
        textObjects: {
            objectList: [
            ],
            createObject(props) {
                let obj = {
                    text: props ? (props.text ? props.text : "template") : "template",
                    color: props ? (props.color ? props.color : "#000") : "#000",
                    fontWeight: props ? (props.fontWeight ? props.fontWeight : 500) : 500,
                    textDecoration: props ? (props.textDecoration ? props.textDecoration : "none") : "none",
                    italic: props ? (props.italic ? props.italic : false) : false,
                    dominantBaseline: props ? (props.dominantBaseline ? props.dominantBaseline : "hanging") : "hanging",
                    textAnchor: props ? (props.textAnchor ? props.textAnchor : "start") : "start",
                    x: props ? (props.x ? props.x : 0) : 0,
                    xUnit: props ? (props.xUnit ? props.xUnit : "px") : "px",
                    y: props ? (props.y ? props.y : 0) : 0,
                    yUnit: props ? (props.yUnit ? props.yUnit : "px") : "px",
                    fontSize: props ? (props.fontSize ? props.fontSize : 16) : 16,
                    id: "t" + state.textObjects.objectList.length
                }
                state.textObjects.objectList = [...state.textObjects.objectList ,obj] ;
                state.canvas.redraw()
                changed = true;
            },
            duplicateObject() {
                let obj = state.textObjects.getObject(state.selectedObject.id)
                let newObj = {
                    text: obj.text,
                    color: obj.color,
                    fontWeight: obj.fontWeight,
                    textDecoration: obj.textDecoration,
                    italic: obj.italic,
                    dominantBaseline: obj.dominantBaseline,
                    textAnchor: obj.textAnchor,
                    x: obj.x,
                    xUnit: obj.xUnit,
                    y: obj.y,
                    yUnit: obj.yUnit,
                    fontSize: obj.fontSize,
                    id: "t" + state.textObjects.objectList.length
                }
                state.textObjects.createObject(newObj)
            },
            getObject(id) {
                changed = true;
                return this.objectList.find(x => x.id === id);
            },
            setList(list) {
                this.objectList = list;
            },
            toString() {
                let str = "";
                this.objectList.forEach(element => {
                    str += (element.text+ " ")
                });
                return str;
            }
        },
        canvas: {
            height: 200,
            width: 400,
            viewbox: true,
            background: "#fff",
            setHeight(x) {
                this.height = x;
            },
            setWidth(x) {
                this.width = x;
            },
            setViewbox(bool) {
                this.viewbox = bool;
            },
            setBackground(x) {
                this.background = x;
            },
            getHeight() {
                return this.height;
            },
            getWidth() {
                return this.width;
            },
            getBackground() {
                return this.background;
            },
            getViewbox() {
                if (this.viewbox) return `0 0 ${this.width} ${this.height}`;
                return "";
            },
            redraw() {
                this.height += 0;
            }
        },
        selectedObject: null,
        selectedObjectElement: null,
        exportMode: false,
        setSelected(elem) {
            selected = true;
            this.selectedObject = this.textObjects.getObject(elem.id);
            this.selectedObjectElement = elem;
        },
        getSelected() {
            if (this.selectedObject == null) return false;
            return {obj: this.selectedObject, elem: this.selectedObjectElement};
        },
        resetSelection() {
            selected = false;
            this.selectedObject = null;
            this.selectedObjectElement = null;
            title = "none";
        },
        getList() {
            return this.textObjects.objectList;
        },
        loadState(_state) {
            this.canvas.setHeight(_state.canvas.height);
            this.canvas.setWidth(_state.canvas.width);
            this.canvas.setViewbox(_state.canvas.viewbox);
            this.canvas.setBackground(_state.canvas.background ? _state.canvas.background : "#fff");
            this.textObjects.setList([]);
            _state.objectList.forEach(element => {
                this.textObjects.createObject(element);
            });
            state.canvas.redraw()
            state.resetSelection()
        },
        getState() {
            let _state = {
                canvas: {
                    height: state.canvas.height,
                    width: state.canvas.width,
                    viewbox: state.canvas.viewbox,
                    background: state.canvas.background,
                },
                objectList: state.textObjects.objectList
            }
            _state = JSON.parse(JSON.stringify(_state))
            return _state;
        },
    }
    function handleKeypress(event) {
        if (state.exportMode) return;
        if (event.code == "KeyZ" && event.ctrlKey && !event.shiftKey) {
            undo()
        }
        if (event.code == "KeyZ" && event.ctrlKey && event.shiftKey) {
            redo()
        }
        if (event.code == "KeyY" && event.ctrlKey) {
            redo()
        }
    }
    let edit = false;
    let past = [];
    let present = [state.getState()];
    let future = [];
    let historyReady = true
    $: if (state != undefined && edit == false && state.exportMode == false && historyReady == true){
        future = []
        if (past.length >= 999) {
            past.push(present[0]);
            past = past.slice(1,past.length)
        } else {
            past.push(present.pop());
            present.push(state.getState());
        }
    } else if (edit == true) {
        edit = false;
    } else if (state.exportMode == true) {
        historyReady = false;
    } else if (state.exportMode == false) {
        historyReady = true;
    }
    function undo() {
        d.style.display = 'none'
        edit = true
        if (past.length == 0) return;
        let newPresent = past.pop()
        let newFuture = present.pop()
        future.push(newFuture)
        present.push(newPresent)
        state.loadState(newPresent)
        console.log(past,present,future);
    }
    function redo() {
        d.style.display = 'none'
        edit = true
        if (future.length == 0) return;
        let newPresent = future.pop()
        let newPast = present.pop()
        present.push(newPresent)
        past.push(newPast)
        state.loadState(newPresent)
    }
    $: selected = false;
    $: title = "none"
    $: changed = false;
    function updateTitle() {
        if (selected) {
            let obj = state.getSelected().obj
            if (obj.text.length > 20) {
                title = obj.text.slice(0,20) +`... (id ${obj.id})`
            } else {
                title = obj.text +` (id ${obj.id})` 
            }
        } else {
            title = "none"
        }
    }
    let selObj = state.getSelected();
    function objClick(event) {
        if (event.shiftKey) return;
        selObj = state.getSelected()
        let selectedObj = state.getSelected().obj;
        if (event.target.matches(".zoom__btn")) return;
        if (event.target.matches(".text__object")) {
            let obj = state.textObjects.getObject(event.target.id)
            if (state.getSelected() == false) {
                state.setSelected(document.getElementById(obj.id));
                event.target.classList.add("selected__obj")
                updateTitle()
                return;
            }
            if (selectedObj.id === obj.id) return;
            state.setSelected(document.getElementById(obj.id));
            updateTitle()
            return
        }
        if (!selectedObj) return;
        state.selectedObjectElement.classList.remove("selected__object")
        state.resetSelection();
        updateTitle()
    }
    function deselect() {
        selObj = null;
        state.resetSelection();
    }
    function _delete() {
        state.textObjects.objectList = state.textObjects.objectList.filter(obj => obj.id != state.selectedObject.id)
        let i = 0;
        state.textObjects.objectList = state.textObjects.objectList.filter(obj => obj.id = "t" + i++) 
        state.resetSelection()
        title = "none"
        changed = true;
    }
    let scale = 1
    function canvZoom(event) {
        d.style.display = 'none'
        event.preventDefault();
        scale += scale * event.deltaY * -0.001;
        scale = Math.min(Math.max(.125, scale), 10)
        let c = document.getElementById("scaler");
        c.style.transform = `scale(${scale})`;
    }
    let templates = [
        {
            canvas: {
                height: 100,
                width: 412,
                viewbox: true,
                background: "#000"
            },
            objectList: [
                {
                    "text": "20% Off Tees - Online Only",
                    "color": "#f28b02",
                    "fontWeight": 500,
                    "textDecoration": "none",
                    "italic": false,
                    "dominantBaseline": "middle",
                    "textAnchor": "middle",
                    "x": 50,
                    "xUnit": "%",
                    "y": 24,
                    "yUnit": "%",
                    "fontSize": 24,
                    "id": "t0"
                },
                {
                    "text": "Shop the Tee Wall",
                    "color": "#fff",
                    "fontWeight": 500,
                    "textDecoration": "none",
                    "italic": false,
                    "dominantBaseline": "middle",
                    "textAnchor": "middle",
                    "x": 50,
                    "xUnit": "%",
                    "y": 58,
                    "yUnit": "%",
                    "fontSize": 40,
                    "id": "t1"
                },
                {
                    "text": "Pick your faves from our trending tees selection",
                    "color": "#fff",
                    "fontWeight": 500,
                    "textDecoration": "none",
                    "italic": false,
                    "dominantBaseline": "middle",
                    "textAnchor": "middle",
                    "x": 50,
                    "xUnit": "%",
                    "y": 86,
                    "yUnit": "%",
                    "fontSize": 12,
                    "id": "t2"
                }
            ],
            "name": "Mobile Banner",
        }
    ]
    function clearCanvas() {
        state.textObjects.objectList = state.textObjects.objectList.filter(obj => !obj.id)
    }
    function openMenu(event) {
        if (!event.target.matches(".btn__menu")) return;
        let btn = event.target.firstChild
        btn.style.background = "#666"
        btn.classList.add("active__button")
        let id = event.target.id;
        let menu = document.getElementById(id+"Menu");
        menu.style.display = "block";   
    }
    function closeMenu(event) {
        if (event.target.parentNode.matches(".btn__menu")) return;
        let btn = document.getElementById(event.target.id).firstChild
        btn.style.background = "#222"
        let id = event.target.id;
        let menu = document.getElementById(id+"Menu");
        menu.style.display = "none";
    }
    let savedStates = [];
    let loaded = false;
    onMount(() => {
        
        if (localStorage.getItem("savedStates")) {
            savedStates = JSON.parse(localStorage.getItem("savedStates"))
        }
        document.addEventListener("keypress", handleKeypress);
        document.addEventListener('mouseover', hover)
    })
    
    function getTime(ts) {
        let time = Math.round((Date.now()-ts)/1000)
        let string = `${Math.round((Date.now()-ts)/1000)} Seconds`;
        let unit = "s"
        if (time > 60) {time = Math.round(time/60); string = `${time} Minutes`; unit = "m"}
        if (time > 60 && unit == "m") {time = Math.round(time/60); string = `${time} Hours`; unit = "h"}
        if (time > 24 && unit == "h") {time = Math.round(time/24); string = `${time} Days`; unit = "d"}
        if (time > 30 && unit == "d") {time = Math.round(time/24); string = `${time} Months`; unit = "mo"}
        if (time > 12 && unit == "mo") {time = Math.round(time/24); string = `${time} Years`; unit = "y"}
        return string;
    }
    let exportedCode = "";
    function _export() {
        if (state.exportMode == true) {
            let svgBody = "";
            ariaDesc = state.textObjects.toString();
            state.textObjects.objectList.forEach(obj => {
                let textbb = document.getElementById(obj.id).getBoundingClientRect();
                let svgbb = document.getElementById("cnvs").getBoundingClientRect();
                let calculatedWidth = (textbb.width/svgbb.width)*state.canvas.width;
                let str = 
                `<text textLength="${calculatedWidth}px" text-decoration="${obj.textDecoration}" font-style="${obj.italic ? "italic" : "none"}" fill="${obj.color}" font-weight="${obj.fontWeight}" dominant-baseline="${obj.dominantBaseline}" text-anchor="${obj.textAnchor}" x="${obj.x+obj.xUnit}" y="${obj.y+obj.yUnit}" font-size="${obj.fontSize+"px"}">${obj.text}</text>`
                svgBody += str;
                ariaDesc += obj.text + " ";
            })
            let vb = state.canvas.viewbox ? `viewbox="0 0 ${state.canvas.width} ${state.canvas.height}"` : `height="${state.canvas.height}" width="${state.canvas.width}"`;
            exportedCode = `<svg ${vb} style="background-color:${state.canvas.background};font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji','Segoe UI Symbol';height: ${svgHeight}" aria-labeled-by="${ariaTitle} ${ariaTitle + "d"}"><title id="${ariaTitle}">${ariaTitle}</title><desc id="${ariaTitle + "d"}">${ariaDesc}</desc>${svgBody}</svg>`
            console.log('updated exported code');
            if (wrapped) {
                exportedCode = `<div style="display:flex;justify-content:center;align-items:center;width:100%;height:${containerHeight}px;background-color:${state.canvas.background}">${exportedCode}</div>`
                console.log(exportedCode);
            }
        }
        transPos={x:0,y:0};
        panTrans="translate3D(0,0,0)";
        scale = 1;
        let svgBody = "";
        ariaDesc = state.textObjects.toString();
        state.textObjects.objectList.forEach(obj => {
            let textbb = document.getElementById(obj.id).getBoundingClientRect();
            let svgbb = document.getElementById("cnvs").getBoundingClientRect();
            let calculatedWidth = (textbb.width/svgbb.width)*state.canvas.width;
            let str = 
            `<text textLength="${calculatedWidth}px" text-decoration="${obj.textDecoration}" font-style="${obj.italic ? "italic" : "none"}" fill="${obj.color}" font-weight="${obj.fontWeight}" dominant-baseline="${obj.dominantBaseline}" text-anchor="${obj.textAnchor}" x="${obj.x+obj.xUnit}" y="${obj.y+obj.yUnit}" font-size="${obj.fontSize+"px"}">${obj.text}</text>`
            svgBody += str;
            ariaDesc += obj.text + " ";
        })
        let vb = state.canvas.viewbox ? `viewbox="0 0 ${state.canvas.width} ${state.canvas.height}"` : `height="${state.canvas.height}" width="${state.canvas.width}"`;
        exportedCode = `<svg ${vb} style="background-color:${state.canvas.background};font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji','Segoe UI Symbol';height: ${svgHeight}" aria-labeled-by="${ariaTitle} ${ariaTitle + "d"}"><title id="${ariaTitle}">${ariaTitle}</title><desc id="${ariaTitle + "d"}">${ariaDesc}</desc>${svgBody}</svg>`
        if (wrapped) {
            exportedCode = `<div style="display:flex;justify-content:center;align-items:center;width:100%;height:${containerHeight}px;background-color:${state.canvas.background}">${exportedCode}</div>`
        }
        state.exportMode = true;
        document.getElementById("scaler").style.transform = "scale(1)"
        
    }
    let dragging = false;
    let moved = false;
    let dragElement;
    let dragObj;
    let dragPos;
    let timeout;
    function startDrag(event) {
        if (event.shiftKey) return
        if (!event.target.matches(".text__object")) return;
        app.addEventListener("mouseup", stopDrag);
        timeout = setTimeout(function(){
            dragPos = [event.pageX,event.pageY]
            dragElement = event.target
            dragObj = state.textObjects.getObject(event.target.id)
            app.addEventListener("mousemove", drag);
            dragging = true;
            clearTimeout(timeout);
            timeout = null;
        }, 25);
    }
    function drag(event) {
        if (!dragging) return;
        moved = true;
        dragElement.style.transform = `translate3D(${(dragPos[0] - event.pageX)*(-1)/(scale)}px,${(dragPos[1] - event.pageY)*(-1)/(scale)}px,0)`
    }
    function stopDrag(event) {
        if (timeout) {clearTimeout(timeout); return;}
        dragging = false;
        dragElement.style.transform = "none"
        app.removeEventListener("mousemove", drag);
        app.removeEventListener("mouseup", stopDrag);
        if (!moved) {return} else {moved = false};
        if (dragObj.xUnit == "%") {
            dragObj.xUnit = "px";
            dragObj.x = state.canvas.width*(dragObj.x/100)
        }
        if (dragObj.yUnit == "%") {
            dragObj.yUnit = "px";
            dragObj.y = state.canvas.height*(dragObj.y/100)
        }
        dragObj.x -= (dragPos[0] - event.pageX)/scale
        dragObj.y -= (dragPos[1] - event.pageY)/scale
        dragPos = []
        dragElement = null;
        dragObj = null;
        state.canvas.height += 0;
    }
    let saves = [];
    function showSaves() {
        let request = window.indexedDB.open("SavesDB", 1)
        request.onerror = (event) => {
            console.error(`Database error: ${event.target.errorCode}`);
        };

        request.onsuccess = (event) => {
            let db = event.target.result;
            let s = state.getState()
            s.name = "saved thing"
            // insertSave(s)
            getSaves(db)
            savesModal = true;
        };
        request.onupgradeneeded = (event) => {
            let db = event.target.result;

            // create the Contacts object store 
            // with auto-increment id
            let store = db.createObjectStore('Saves', {
                keyPath: 'id'
            });
            

            // create an index on the email property
            let index = store.createIndex('id', 'id', {
                unique: true
            });
        };
    }
    function insertSave(p, save) {
        let request = window.indexedDB.open("SavesDB", 1)
        request.onerror = (event) => {
            console.error(`Database error: ${event.target.errorCode}`);
        };

        request.onsuccess = (event) => {
            let db = event.target.result;
            // create a new transaction
            let txn = db.transaction('Saves', 'readwrite');

            // get the object store
            let store = txn.objectStore('Saves');
            //
            let s;
            if (!save) {
                s = state.getState()
            } else {
                s = save
            }
            s.name = newSaveName;
            s.id = Date.now();
            let query = store.put(s);

            // handle success case
            query.onsuccess = function (event) {
                // console.log(event);
            };

            // handle the error case
            query.onerror = function (event) {
                console.log(event.target.errorCode);
            }

            // close the database once the 
            // transaction completes
            txn.oncomplete = function () {
                getSaves(db);
                db.close();
            };
        };
    }
    function deleteSave(id) {
        let request = window.indexedDB.open("SavesDB", 1)
        request.onerror = (event) => {
            console.error(`Database error: ${event.target.errorCode}`);
        };

        request.onsuccess = (event) => {
            let db = event.target.result;
            // create a new transaction
            let txn = db.transaction('Saves', 'readwrite');

            // get the object store
            let store = txn.objectStore('Saves');
            //
            let query = store.delete(id);

            // handle success case
            query.onsuccess = function (event) {
                // console.log(event);
            };

            // handle the error case
            query.onerror = function (event) {
                console.log(event.target.errorCode);
            }

            // close the database once the 
            // transaction completes
            txn.oncomplete = function () {
                getSaves(db);
                db.close();
            };
        };
    }
    function getSaves(db) {
        let txn = db.transaction('Saves', 'readonly');
        let store = txn.objectStore('Saves');

        let query = store.getAll();

        query.onsuccess = (event) => {
            if (!event.target.result) {
                console.log(`The contact with ${id} not found`);
            } else {
                saveMenuNew = false;
                saves = event.target.result;
            }
        };

        query.onerror = (event) => {
            console.log(event.target.errorCode);
        }

        txn.oncomplete = function () {
            db.close();
        };
    };
    let savesModal = false;
    let saveMenuNew = false;
    let newSaveName;
    let panPos;
    let transPos = {x: 0, y: 0};
    let panTrans = `translate3D(0,0,0)`
    function pan(event) {
        if (event.shiftKey) {
            event.preventDefault()
            panPos = {x: event.clientX,y: event.clientY}
            document.addEventListener("mousemove", panMove);
            document.addEventListener("mouseup", panEnd);
        }
    }
    function panMove(event) {
        event.preventDefault()
        let c = document.getElementById("cnvs");
        if (state.exportMode) {
            c = document.querySelector(".preview__canvas");
        }
        let dx = (event.clientX - panPos.x)/scale
        let dy = (event.clientY - panPos.y)/scale
        panTrans = `translate3D(${transPos.x + dx}px,${transPos.y + dy}px,0)`
        c.style.transform = `${panTrans}`
    }
    function panEnd(event) {
        document.removeEventListener("mousemove", panMove);
        document.removeEventListener("mouseup", panEnd);
        transPos.x += parseInt((event.clientX - panPos.x)/scale);
        transPos.y += parseInt((event.clientY - panPos.y)/scale);
        panTrans = `translate3D(${transPos.x}px,${transPos.y }px,0)`;
        c.style.transform = `${panTrans}`;
    }
    let modal;
    let modalPos;
    let modalOffset = {x:0, y:0};
    function modalDrag(event) {
        if (event.target.matches(".action__bar")) {
            modal = event.target.parentNode;
            modalPos = {x: event.clientX, y: event.clientY};
            if (modal.style.transform) {
                let arr = modal.style.transform.match(/([^e,^\(])\d{1,4}/g);
                modalOffset.x = parseInt(arr[0]); modalOffset.y = parseInt(arr[1]);
            } else {
                modalOffset = {x:0, y:0};
            }
            document.addEventListener("mousemove", modalMove);
            document.addEventListener("mouseup", modalStop);
        }
    }
    function modalMove(event) {
        event.preventDefault()
        let dx = event.clientX - modalPos.x 
        let dy = event.clientY - modalPos.y
        modal.style.transform = `translate3D(${modalOffset.x + dx}px,${modalOffset.y + dy}px,0)`
    }
    function modalStop(event) {
        document.removeEventListener("mousemove", modalMove);
        document.removeEventListener("mouseup", modalStop);
        let rect = modal.getBoundingClientRect()
        modalOffset.x += parseInt(event.clientX - modalPos.x);
        modalOffset.y += parseInt(event.clientY - modalPos.y);
        if (rect.x < (-1*(rect.width/8)) || rect.x > document.documentElement.clientWidth-(rect.width*.6) || rect.y < 60 || rect.y > document.documentElement.clientHeight-(rect.height*0.4)) {
            savesModal = false
        }
        modal.style.transform = `translate3D(${modalOffset.x}px,${modalOffset.y}px,0)`
    }
    function download(filename, text) {
        let ft = filename.slice(filename.length-4,filename.length).replace(".",'')
        if ( ft != "svg" && ft != "html") {

            let svg = document.querySelector(".preview__canvas > svg")
            var copy = svg.cloneNode(true);
            var canvas = document.createElement("canvas");
            canvas.width = state.canvas.width;
            canvas.height = state.canvas.height;
            var ctx = canvas.getContext("2d");
            ctx.clearRect(0, 0, state.canvas.width, state.canvas.height);
            var data = (new XMLSerializer()).serializeToString(copy);
            var DOMURL = window.URL || window.webkitURL || window;
            var img = new Image();
            var svgBlob = new Blob([data], {type: "image/svg+xml;charset=utf-8"});
            var url = DOMURL.createObjectURL(svgBlob);
            img.onload = function () {
                ctx.drawImage(img, 0, 0);
                DOMURL.revokeObjectURL(url);
                if (typeof navigator !== "undefined" && navigator.msSaveOrOpenBlob)
                {
                    var blob = canvas.msToBlob();         
                    navigator.msSaveOrOpenBlob(blob, fileName);
                } 
                else {
                    var imgURI = canvas
                        .toDataURL(`image/${ft}`, 1)
                    var element = document.createElement('a');
                    element.setAttribute('href', imgURI);
                    element.setAttribute('download', filename);

                    element.style.display = 'none';
                    document.body.appendChild(element);

                    element.click();

                    document.body.removeChild(element);
                }
                document.removeChild(canvas);
            };
            img.src = url;
            return;
        }
        var element = document.createElement('a');
        element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
        element.setAttribute('download', filename);

        element.style.display = 'none';
        document.body.appendChild(element);

        element.click();

        document.body.removeChild(element);
    }
    function copy(event) {
        clipboardCopy(exportedCode);
        event.target.textContent = "Copied"
        setTimeout(() =>{
            event.target.textContent = "Copy SVG to Clipboard"
            clearTimeout();
        }, 1000)
    }
    let clipboardCopySupported = true;
    let visuallyHidden = 'position:absolute;height:1px;width:1px;overflow:hidden;clip:rect(1px,1px,1px,1px)';
    function clipboardCopy(text) {
        if (!clipboardCopySupported) return false;
        let txt = document.createElement('textarea');
        txt.textContent = text;
        txt.setAttribute('style', visuallyHidden);
        document.body.appendChild(txt);
        txt.select();
        try {
            console.log("trying to copy")
            return document.execCommand('copy');
        } catch (ex) {
            return false;
        } finally {
            document.body.removeChild(txt);
        }
    }

    let previewHeight = 600;
    let previewWidth = 1200;
    let selectedMobile;
    let mobileDevies = {"iphonese":[375,667],"iphonexr":[414,896],"iphone12pro":[390,844],"pixel5":[393,851],"samsunggalaxys8":[360,740],"samsunggalaxys20ultra":[412,915]}
    let selectedTablet;
    let tabletDevies = {"ipadair":[820,1180],"ipadmini":[768,1024],"surfacepro":[912,1368]}
    let responsive = "none";
    function updatePreview(p) {
        transPos={x:0,y:0};
        panTrans="translate3D(0,0,0)";
        scale = 1;
        if (p == "m") {
            if (selectedMobile != "none") {
                selectedTablet = "none";
                let device = mobileDevies[selectedMobile];
                previewHeight = device[1];
                previewWidth = device[0];
            }
        }
        if (p == "t") {
            if (selectedTablet != "none") {
                selectedMobile = "none";
                let device = tabletDevies[selectedTablet];
                previewHeight = device[1];
                previewWidth = device[0];
            }
        }
        if (p == "r") {
            selectedTablet = "none";
            selectedMobile = "none";
        }
        if (p == "d") {
            selectedTablet = "none";
            selectedMobile = "none";
            previewHeight = 600;
            previewWidth = 1200
        }
        document.getElementById("scaler").style.transform = "scale(1)"
        document.querySelector(".preview__canvas").style.transform = panTrans
    }
    let d;
    let t;
    function hover(event) {
        if (event.target.matches(".text__object")) {
            t = event.target;
            document.querySelectorAll(".text__object").forEach(e =>{
                e.style.pointerEvents = 'none'
            })
            t.style.pointerEvents = 'all'
            document.removeEventListener('mouseover', hover)
            document.addEventListener('mousemove', move)
            t.addEventListener('mouseleave', exit)
        }
    }
    function exit(event) {
        document.addEventListener('mouseover', hover)
        document.removeEventListener('mousemove', move)
        t.removeEventListener('mouseleave', exit)
        document.querySelectorAll(".text__object").forEach(e =>{
            e.style.pointerEvents = 'all'
        })
        d.style.display = 'none'
    }
    function move(event) {
        if (!d.style) return;
        let bb = t.getBoundingClientRect()
        d.style.top = bb.y + "px";
        d.style.left = bb.x + "px";
        d.style.width = bb.width + "px"
        d.style.height = bb.height + "px"
        d.style.display = "block"
    }
    let downloadModal = false;
    let fileName;
    let fileType;
    function onbeforeunload(event) {
        event.preventDefault();
        let _s = state.getState()
        let _state = {
            ts: Date.now(),
            canvas: _s.canvas,
            objectList: _s.objectList
        }
        savedStates = [_state, ...savedStates.splice(0,9)]
        if (changed && _state.objectList.length > 0) localStorage.setItem("savedStates", JSON.stringify(savedStates));
        event.returnValue = '';
        return '...';
    }
    let helpModal = false;
    let helpRef = ["Intro","Canvas","Text Objects","Keybinds","Saves","Export","Templates"]
    function scrollTo(ref) {
        document.getElementById(ref.toLowerCase()).scrollIntoView()
        document.querySelector(".help__modal").scrollTo(0,0)
    }
    let ariaTitle;
    let ariaDesc;
    let wrapped = false;
    let svgHeight = "unset"
    let containerHeight = 100;
    function wrapExport() {
        // exportedCode = `<div style="display:flex;justify-content:center;align-items:center;width:100%;height:100px;background:${state.canvas.background}">${exportedCode}</div>`
        svgHeight =  svgHeight == "100%" ? "unset" : "100%";
        wrapped = !wrapped;
        _export()
    }
</script>
<svelte:window on:beforeunload={onbeforeunload}/>
<div class="styles">
    <style id="checkbox">
        /* Customize the label (the container) */
        .container {
        display: block;
        position: relative;
        padding-left: 35px;
        margin-bottom: 12px;
        cursor: pointer;
        font-size: 22px;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        }
    
        /* Hide the browser's default checkbox */
        .container input {
        position: absolute;
        opacity: 0;
        cursor: pointer;
        height: 0;
        width: 0;
        }
    
        /* Create a custom checkbox */
        .checkmark {
        position: absolute;
        top: 0;
        left: 0;
        height: 22px;
        width: 22px;
        background-color: #555;
        }
    
        /* On mouse-over, add a grey background color */
        .container:hover input ~ .checkmark {
        background-color: #ccc;
        }
    
        /* When the checkbox is checked, add a blue background */
        .container input:checked ~ .checkmark {
        background-color: #999;
        }
    
        /* Create the checkmark/indicator (hidden when not checked) */
        .checkmark:after {
        content: "";
        position: absolute;
        display: none;
        }
    
        /* Show the checkmark when checked */
        .container input:checked ~ .checkmark:after {
        display: block;
        }
    
        /* Style the checkmark/indicator */
        .container .checkmark:after {
        left: 9px;
        top: 5px;
        width: 5px;
        height: 10px;
        border: solid white;
        border-width: 0 3px 3px 0;
        -webkit-transform: rotate(45deg);
        -ms-transform: rotate(45deg);
        transform: rotate(45deg);
        }
    </style>
</div>
<style>
    .text__object {
        -webkit-user-select: none; /* Safari */        
        -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* IE10+/Edge */
        user-select: none; /* Standard */
    }
    button, select {
        cursor: pointer;
    }
    .application {
        box-sizing: border-box;
        width: 100%;
        height: 100%;
        background-color: #c2c2c2;
        position: fixed;
        top: 0px;
        left: 0;
        z-index: 1000;
        display: grid;
        grid-template-rows: 100px 1fr;
        border: 2px solid #000;
        overflow: hidden;
        color: #fff;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
          Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji",
          "Segoe UI Symbol";
    }
    .canvas__container {
        display: flex;
        position: relative;
        justify-content: center;
        align-items: center;
        background-color: #555;
        z-index: 5;
        overflow: hidden;
    }
    .canvas {
        background-color: #fff;
    }
    .controls {
        background-color: #333;
        z-index: 10;
    }
    .controls__menu {
        width: 100%;
        height: 30px;
        background-color: #222;
        display: flex;
    }
    .btn__menu > button {
        width: fit-content;
        height: 100%;
        border: none;
        color: #fff;
        background-color: #222;
    }
    .options {
        display: flex;
        gap: 5px;
        height: calc(100% - 30px);
    }
    .btn__menu {
        position: relative;
        width: fit-content;
        margin: 0;
        z-index: 1001;
    }
    .options > button {
        height: 100%;
        border: none;
        background: none;
        color: #fff;
    }
    .options > button {
        height: 100%;
        border: none;
        background: none;
        color: #fff;
    }
    .menu__panel {
        position: absolute;
        display: none;
        min-width: 100px;
        max-width: 300px;
        width: max-content;
    }
    .menu__panel > button {
        text-align: left;
        width: 100%;
        height: 100%;
        border: none;
        color: #fff;
        background-color: #666;
    }
    .menu__panel > button:hover {
        border: none;
        color: #fff;
        background-color: #444;
    }
    .menu__panel > button:active {
        border: none;
        color: #fff;
        background-color: #777;
    }
    .options__input {
        display: flex;
        align-items: end;
        justify-content: center;
        flex-direction: column;
        border-right: 1px solid #222;
        padding: 0 5px;
    }
    input, select {
        background-color: #555;
        border: 1px solid #333;
        outline: none !important;
        width: 50px;
        height: 28px;
        color: #fff;
    }
    input {
        text-align: end;
    }
    select {
        width: 40px
    }
    .wide {
        width: 80px;
    }
    input::-webkit-outer-spin-button,
    input::-webkit-inner-spin-button{
        -webkit-appearance: none;
        margin: 0;
    }
    .selected__obj {
        text-decoration: underline;
    }
    .trans_btns {
        position: fixed;
        bottom: 0;
        height: 100%;
        width: 100%;
        display: flex;
        justify-content: end;
        align-items: flex-end;
        pointer-events: none;
    }
    .zoom__btn {
        border: none;
        background: #333;
        color: #fff;
        border-radius: .3rem;
        margin: 5px 5px 5px 0px;
        pointer-events: all;
    }
    .zoom__btn:hover {
        background: #222;
    }
    .zoom__btn:active {
        background: #888;
    }
    .zoom-text {
        position: absolute;
        left: 5px;
        bottom: 5px;
    }
    .recenter__btn {
        border: none;
        background: #333;
        color: #fff;
        border-radius: .3rem;
        margin: 5px 5px 5px 0px;
        pointer-events: all;
    }
    .recenter__btn:hover {
        background: #222;
    }
    .recenter__btn:active {
        background: #888;
    }
    .text__area {
        position: absolute;
        background-color: #333;
        top: 70px;
        width: 100%;
        padding: 3px 6px;
        border-top: 1px solid #222;
    }
    .option__btn:hover {
        background-color: #444;
    }
    .option__btn:active {
        background-color: #666;
    }
    .close--btn {
        position: absolute;
        top: 0;
        right: 0;
        height: 24px;
        width: 24px;
        display: flex;
        align-items: center;
        justify-content: center;
        background: #333;
        color: #e00;
    }
    .saves__modal {
        display: block;
        position: absolute;
        top: calc(50% - 300px);
        left: calc(50% - 300px);
        height: 600px;
        width: 600px;
        background: #333;
        z-index: 10000;
        border-radius: 10px;
        overflow: hidden;
    }
    .action__bar {
        width: 100%;
        height: 20px;
        display: flex;
        align-items: center;
        background-color: #111;
        border-radius: 10px 10px 0 0;
    }
    .action__close {
        border-radius: 50%;
        margin: 4px;
        border: none;
        height: 12px;
        width: 12px;
        padding: 0;
        background: #e00;
    }
    .action__close:hover {
        background: #faa;
    }
    .saves__container {
        height: calc(100% - 60px);
        width: 100%;
        overflow: auto;
    }
    .saves__container::-webkit-scrollbar {
        background-color: #2a2a2a;
    }
    .saves__container::-webkit-scrollbar-thumb {
        background-color: #6b6b6b;
    }
    .save__entry {
        width: 100%;
        height: fit-content;
        max-height: 250px;
        border: 2px solid #111;
        margin: 8px 0;
    }
    .save__actions {
        width: 100%;
        margin: 8px 0;
        display: grid;
        gap: 8px;
        grid-template-columns: 1fr 100px 100px 100px;
    }
    .save__preview {
        max-height: 200px;
        border: 1px;
    }
    .svg__preview {
        height: 100%;
        max-height: 200px;
    }
    .load__action {
        background-color: #00b;
        border-radius: 3px;
        border: none;
        color: #fff;
        font-weight: 600;
    }
    .delete__action {
        background-color: #c00;
        border-radius: 3px;
        border: none;
        color: #fff;
        font-weight: 600;
    }
    .export__action {
        background-color: #0a0;
        border-radius: 3px;
        border: none;
        color: #fff;
        font-weight: 600;
    }
    .entry__name {
        margin-left: 8px;
    }
    .save__menu {
        background-color: #2d2d2d;
        height: fit-content;
    }
    .create__save {
        background-color: #00a;
        border-radius: 3px;
        border: none;
        color: #fff;
        font-weight: 600;
        padding: 8px;
    }
    .input__box {
        display: grid;
        grid-template-columns: 100px 1fr 100px;
    }
    .export__modal {
        display: none;
        position: absolute;
        top: 100px;
        left:0;
        height:100%;
        width:100%;
        background: #333;
        z-index: 1000;
        overflow: hidden;
    }
    .ex-btn {
        background-color: #333;
        border: none;
        color: #fff;
        border-radius: 3px;
        margin: 4px;
        padding: 6px 12px;
    }
    .export__area {
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        overflow: auto;
        height: 100%;
    }
    .export__btns {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        background-color: #222;
    }
    .preview {
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .preview__area {
        background-color: #555;
        width: 100%;
        height: 100%;
    }
    .preview__canvas {
        background-color: #c2c2c2;
    }
    #scaler {
        height: 100%;
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .modal__btn {
        background-color: #444;
        border: none;
        border-radius: 3px;
        color: #fff;
        padding: 4px 8px;
        width: 96%;
        margin: 8px 2%;
    }
    .help__modal {
        display: block;
        position: absolute;
        top: calc(50% - 300px);
        left: calc(50% - 500px);
        height: 700px;
        width: 1000px;
        background: #333;
        z-index: 10000;
        border-radius: 10px;
        overflow: hidden;
    }
    .help__container {
        width: 100%;
        height: 610px;
        display: grid;
        grid-template-columns: 180px 1fr;
    }
    .help__sidebar::-webkit-scrollbar, .help__content::-webkit-scrollbar {
        background-color: #2a2a2a;
    }
    .help__sidebar::-webkit-scrollbar-thumb, .help__content::-webkit-scrollbar-thumb {
        background-color: #6b6b6b;
    }
    .help__header {
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        border-bottom: 1px solid;
        border-top: 1px solid;
    }
    .help__sidebar {
        border-right: 1px solid;
        overflow: auto;
        display: flex;
        flex-direction: column;
    }
    .sidebar__item {
        height: fit-content;
        width: 100%;
        border-bottom: 1px solid;
        text-align: center;
    }
    .sidebar__item > a {
        color: #fff !important;
        text-decoration: none;
    }
    .help__content {
        overflow: auto;
        padding: 0 8px;
    }
    .help__content > p {
        max-width: unset;
        font-weight: 200;
    }
    .config__menu {
        position: absolute;
        top: 105px;
        left: 0;
        background-color: #333;
        padding: 4px 8px;
        width: 250px;
        z-index: 99;
    }
    textarea {
        max-width: 230px;
        min-width: 229px;
        max-height: 300px;
        background-color: #555;
        border: 1px solid #333;
        outline: none !important;
        color: #fff;
    }
</style>
<div class="hoverBox" style="display:none;border-radius:3px;pointer-events:none;border:2px solid #53f;position:absolute;z-index:9999999;height:40px;width:100px;transform:scale(1.01);top:0;left:0;" bind:this={d}></div>
<div class="application" bind:this={app} on:mousedown={startDrag} style='font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";'>
    {#if helpModal}
    <div class="help__modal">
        <div class="action__bar" on:mousedown={modalDrag}>
            <button class="action__close" on:click={()=>{helpModal = false;}}></button>
            <span style="display:block;margin:auto;transform:translate3D(-16px,-3px,0);pointer-events:none">Help</span>
        </div>
        <div class="help__header">
            <h2>SVG Text Builder Guide</h2>
        </div>
        <div class="help__container">
            <div class="help__sidebar">
                {#each helpRef as ref}
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <div on:click={()=>{scrollTo(ref)}} class="sidebar__item" data-ref={`${ref.toLowerCase()}`}>{ref}</div>
                {/each}
            </div>
            <div class="help__content">
                <h3 id="intro">Introduction</h3>
                <p>
                    Welcome to the text builder help guide! This is a tool that lets you create typographical elements that scale perfectly to any device. You could use this to make banners, cards, badges, or anything anything that uses text.
                </p>
                <p>
                    There are many advantages to using SVG's instead of only HTML. Making elements scale to all device sizes can be time-consuming and error-prone. A banner that looks perfect on desktop could end up wrapping on mobile, increasing the height of the banner and wasting precious vertical space. This is where SVG's come in handy. SVG stands for "Scalar Vector Graphic", it is instructions for the browser on how to create and render an image. They are massively more performant than using images because no network calls need to be made, and the file is a fraction of the size. What makes them useful for text is that they scale the same way images do. This means you can have a banner that is perfectly laid out and readable on desktop that looks identical on mobile. It is a one-size fits all solution. Continue reading to learn how to use this tool!
                </p>
                <h3 id="canvas">The Canvas</h3>
                <p>
                    The canvas is where your project is created. It has a few properties you can configure to meet your needs.
                </p>
                <p>
                    Viewbox: The first important property to understand about the canvas is its viewbox. This is essentially the aspect ratio of the SVG, if it has a viewbox of 200x400 (width, height) then the aspect ratio would be 1:2 (200/400 = 0.5 or 1/2). If you don't know what aspect ratio is, it just means the height will always scale to be the same size relative to the width. If there is no viewbox, then the SVG size will be equal to its height and width and will not scale across devices. To get a better understanding, try exporting an SVG with viewbox on and off.
                </p>
                <p>
                    Canvas Height and Width: These values are used to set the exact size of the SVG if viewbox is disabled, or the viewbox size if viewbox is enabled. The units are pixels (px), but you don't need to enter that part.
                </p>
                <p>
                    Background Color: This sets the background color of the SVG. You can enter colors in many different ways. The simplest is to type out the color name, such as "red" or "magenta". You can also use hex values like "#fff" and "#0080ff", or RGB and RGBA values such as "rgb(255,0,0)" or "rgba(0,255,100,0.5)".
                </p>
                <h3 id="text objects">Text Objects</h3>
                <p>
                    Setting up a canvas is great, now it's time to put some content on it. To do this, click the "+" button on the far left of the menu bar. This will add a text object into the canvas that says "template". These text objects are only visible when on the canvas, and anything that goes off the edge will be hidden (Try not to do that). In order to configure this new text object, click on it to select it. Once selected, you will notice the menu bar will change to display properties of that object. Below is what they all do, starting from the left.
                </p>
                <p>
                    Deselect: Deselects the selected object.
                </p>
                <p>
                    Object X and Y: These properties have two different input values, the value and unit. The value is like the coordinates you know, but when it comes to computer graphics the origin (0,0) is at the top left corner rather than the center. There are two units to choose from, "px" and "%", "px" makes the value move the object to exactly those pixel coordinates (example x=10px moves the object 10px to the right), and "%" positions it at a percentage of the SVG's size (example x=50% move the object to exactly the middle or 50% of the SVG's width). The way these values position the object is affected by other properties (text anchor and baseline), so keep reading if you are confused about the objects placement. You can also click and drag text objects to change these values, although it will convert the units back to "px".
                </p>
                <p>
                    Text Anchor: This property determines where the horizontal (x) origin of the objects position is calculated. As a default, the origin will start at the top left of the object. Text anchor changes this depending on the selected value. Start sets the x origin to the beginning or left of the object. Middle sets the x origin to the center of the object. End sets the x origin to the end of the object.
                </p>
                <p>
                    Baseline: This property determines where the vertical (y) origin of the objects position is calculated. As a default, it starts at the top left corner of the object. Baseline changes this depending on the selected value. Top sets the y origin to the top of the object. Center sets the y origin to the center of the object. Bottom sets the y origin to the bottom of the object. 
                </p>
                <p>
                    Font Size: This property changes the size of the font.
                </p>
                <p>
                    Font Weight: This property changes the weight of the font. If you are unfamiliar with what font weight is, it is basically how thick or bold the font is.
                </p>
                <p>
                    Font Color: This property changes the color of the text objects font. You can enter colors in many different ways. The simplest is to type out the color name, such as "red" or "magenta". You can also use hex values like "#fff" and "#0080ff", or RGB and RGBA values such as "rgb(255,0,0)" or "rgba(0,255,100,0.5)".
                </p>
                <p>
                    Text Decoration: This property is for adding a line through or underline.
                </p>
                <p>
                    Italic: Makes the font italic.
                </p>
                <p>
                    Delete: Removes/Deletes the selected text object from the canvas.
                </p>
                <p>
                    Duplicate: Duplicates the selected text object with all the same properties. At first it doesn't look like it did anything when you click it because the duplicate is exactly on top of the original, so if you move it or change a property you will see it's there.
                </p>
                <p>
                    Text Content: This property changes the text content of the object. Whatever you type in this box will be in the text object on the canvas. You can use any kind of character, letters, numbers, special characters, emojis, anything.
                </p>
                <h3 id="keybinds">Keybinds</h3>
                <p>
                    There are some keyboard shortcuts that can help you when working in this tool.
                </p>
                <p>
                    Undo: CTRL + Z, this will undo recent edits, up to 999 past changes.
                </p>
                <p>
                    Redo: CTRL + Y, or CTRL + SHIFT + Z, this will redo edits that you undid.
                </p>
                <p>
                    Pan: SHIFT + Click + Move mouse, this will allow you to pan around the canvas (There is a button at the bottom left to re-center)
                </p>
                <p>
                    Zoom: Scroll, this will zoom in and out. (There is a button at the bottom left to reset zoom)
                </p>
                <h3 id="saves">Saves</h3>
                <p>
                    There are two types of saves, manual and auto. Autosaves occur when you close the tab or reload the page, and it will save the last 10 projects. Autosaves can be loaded by hovering on the "Auto-Saves" menu button at the very top of the tool. Manual saves can be created by clicking on the "Saves" menu button at the very top of the page. This will open a saves window which you can move around by dragging. In this window, you can see a list of all saves you created, as well as a "Create a new Save" button. The "Create a new Save" button will prompt you to enter a save name, and hitting "Save" will commit the save to your list.
                </p>
                <p>
                    All saves are local to your browser, so if you saved on chrome and open safari, it won't be there. Don't rely on this feature too much, as local storage can be cleared accidentally, resulting in a loss of all saves. A more permanent way to save projects is to export them, but exported projects cannot be re-imported (Yet. Sorry, this is a difficult feature to implement and may never see the light of day.)
                </p>
                <h3 id="export">Export</h3>
                <p>
                    Hitting the export button, which is found on the menu bar when nothing is selected, will bring you to the export area. The export area allows you to preview the SVG on multiple devices and screen sizes, as well download or copy the SVG. Downloading gives you several options. You can download the SVG as a .svg, .html, .png, .jpeg, or .webp. Or, if you want to just paste it right into site manager or monetate, you can hit the "Copy Code to Clipboard" button, which does exactly that.
                </p>
                <h3 id="templates">Templates</h3>
                <p>
                    The templates top menu button has some presets I created for common uses, such as banners or badges.
                </p>
            </div>
        </div>
    </div>
    {/if}
    {#if savesModal}
    <div class="saves__modal">
        <div class="action__bar" on:mousedown={modalDrag}>
            <button class="action__close" on:click={()=>{savesModal = false;}}></button>
            <span style="display:block;margin:auto;transform:translate3D(-16px,-3px,0);pointer-events:none">Saves</span>
        </div>
        <div class="save__menu">
            {#if !saveMenuNew}
            <div style="width:100%;height:100%;display:flex;justify-content:center;align-items:center"><button class="create__save" on:click={()=>{saveMenuNew = !saveMenuNew}}>Create a new Save</button></div>
            {:else}
            <div class="input__box">
                <span>Save Name: </span>
                <input type="text" style="width: 100%" bind:value={newSaveName}>
                <button class="load__action" on:click={insertSave}>Save</button>
            </div>
            {/if}
        </div>
        <div class="saves__container">
            {#each saves as s}
            <div class="save__entry">
                <div class="save__actions">
                    <span class="entry__name">{s.name}</span>
                    <button class="export__action" on:click={()=>{}}>Export</button>
                    <button class="load__action" on:click={()=>{state.loadState(s)}}>Load</button>
                    <button class="delete__action" on:click={()=>{deleteSave(s.id)}}>Delete</button>
                </div>
                <div class="save__preview">
                    <svg class="svg__preview" viewbox={`0 0 ${s.canvas.width} ${s.canvas.height}`} style="background-color: {s.canvas.background}">
                        {#each s.objectList as obj}
                            <text text-decoration={obj.textDecoration} font-style={obj.italic ? "italic" : "none"} fill={obj.color} font-weight={obj.fontWeight} dominant-baseline={obj.dominantBaseline} text-anchor={obj.textAnchor} x={obj.x+obj.xUnit} y={obj.y+obj.yUnit} font-size={obj.fontSize+"px"}>{obj.text}</text>
                        {/each}
                    </svg>
                </div>
            </div>
            {/each}
        </div>
    </div>
    {/if}
    {#if downloadModal}
    <div class="saves__modal" style="width:350px; height: fit-content">
        <div class="action__bar" on:mousedown={modalDrag}>
            <button class="action__close" on:click={()=>{downloadModal = false;}}></button>
            <span style="display:block;margin:auto;transform:translate3D(-16px,-3px,0);pointer-events:none">Download</span>
        </div>
        <div class="saves__container">
            <div class="input__box" style="display:flex;align-items:center;justify-content:center; margin: 8px 0">
                <span>File Name: </span>
                <input type="text" style="width: fit-content" bind:value={fileName}>
                <select name="" class="wide" bind:value={fileType}>
                    <option value="svg">.svg</option>
                    <option value="html">.html</option>
                    <option value="png">.png</option>
                    <option value="jpeg">.jpeg</option>
                    <option value="webp">.webp</option>
                </select>
            </div>
            <button class="modal__btn" on:click={()=>{download(`${fileName}.${fileType}`, exportedCode)}}>Download</button>
        </div>
    </div>
    {/if}
    <div class="confirm__modal" style="display:none">

    </div>
    <div class="controls">
        <div class="controls__menu" style="position:relative">
            {#key title}
                {#if state.exportMode}
                    <span style="position:absolute; width:100%; height:100%;top:0;right:0; display:flex;align-items:center;justify-content:center" class="selected__title">Exporting</span>
                {:else if state.previewMode}
                    <span style="position:absolute; width:100%; height:100%;top:0;right:0; display:flex;align-items:center;justify-content:center" class="selected__title">Previewing</span>
                {:else}
                    <span style="position:absolute; width:100%; height:100%;top:0;right:0; display:flex;align-items:center;justify-content:center" class="selected__title">Selected: {title}</span>
                {/if}
            {/key}
            <div class="btn__menu" id="file" on:mouseenter={openMenu} on:mouseleave={closeMenu}>
                <button>File</button>
                <div class="menu__panel" id="fileMenu" on:mouseenter={openMenu} on:mouseleave={closeMenu}>
                    <button on:click={clearCanvas}>Clear</button>
                </div>
            </div>
            <div class="btn__menu" id="templates" on:mouseenter={openMenu} on:mouseleave={closeMenu}>
                <button>Templates</button>
                <div class="menu__panel" id="templatesMenu">
                    {#each templates as t}
                        <button on:click={(state.loadState(t))}>{t.name}</button>
                    {/each}
                </div>
            </div>
            <div class="btn__menu" id="states" on:mouseenter={openMenu} on:mouseleave={closeMenu}>
                <button>Auto-Saves</button>
                <div class="menu__panel" id="statesMenu">
                    {#each savedStates as s}
                        <button on:click={state.loadState(s)}>Save from {getTime(s.ts)} ago</button>
                    {/each}
                </div>
            </div>
            <div class="btn__menu" id="statesDB" on:mouseenter={openMenu} on:mouseleave={closeMenu}>
                <button on:click={()=>{showSaves()}}>Saves</button>
                <div class="menu__panel" id="statesDBMenu">
                </div>
            </div>
            <div class="btn__menu" id="help" on:mouseenter={openMenu} on:mouseleave={closeMenu}>
                <button on:click={()=>{helpModal = true}}>Help</button>
                <div class="menu__panel" id="help">
                </div>
            </div>
        </div>
        {#if !state.exportMode}
            {#if !selected}
            <div class="options" id="unselected">
                <button class="option__btn" style="font-size:60px;line-height:0; width: 70px;" on:click={()=>{state.textObjects.createObject()}}>+</button>
                <div class="options__input">
                    <div class="input__container">
                        <span>Canvas Height:</span>
                        <input type="number" bind:value={state.canvas.height} min="0">
                    </div>
                    <div class="input__container">
                        <span>Canvas Width:</span>
                        <input type="number" bind:value={state.canvas.width} min="0">
                    </div>
                </div>
                <div class="options__input">
                    <div class="input__container" style="display:flex">
                        <span>Viewbox:&MediumSpace;</span>
                        <label class="container">
                            <input type="checkbox" bind:checked={state.canvas.viewbox} style="display:none">
                            <span class="checkmark"></span>
                        </label>
                    </div>
                </div>
                <div class="options__input">
                    <div class="input__container">
                        <span>Background Color:</span>
                        <input type="text" style="width: 100px" bind:value={state.canvas.background} on:change={() =>{if (state.canvas.background == null) state.canvas.color = "#fff"}}>
                    </div>
                </div>
                <button class="option__btn" on:click={_export}>Export</button>
            </div>
            {:else}
                {#key selObj}
                    <div class="options" id="selected" style="position:relative">
                        <button class="option__btn" style="border-right: 1px solid #222" on:click={deselect}>Deselect</button>
                        <div class="options__input">
                            <div class="input__container">
                                <span>Object X:</span>
                                <input type="number" bind:value={state.selectedObject.x} min="0" on:change={() =>{if (state.selectedObject.x == null) state.selectedObject.x = 0}}>
                                <select name="" id="" bind:value={state.selectedObject.xUnit} on:change={()=>{if (state.selectedObject.xUnit == "px") {state.selectedObject.x = state.canvas.width*(state.selectedObject.x/100)}; if (state.selectedObject.xUnit == "%") {state.selectedObject.x = ((state.selectedObject.x)/state.canvas.width)*100}}}>
                                    <option value="px">px</option>
                                    <option value="%">%</option>
                                </select>
                            </div>
                            <div class="input__container">
                                <span>Object Y:</span>
                                <input type="number" bind:value={state.selectedObject.y} min="0" on:change={() =>{if (state.selectedObject.y == null) state.selectedObject.y = 0}}>
                                <select name="" id="" bind:value={state.selectedObject.yUnit} on:change={()=>{if (state.selectedObject.yUnit == "px") {state.selectedObject.y = state.canvas.height*(state.selectedObject.y/100)}; if (state.selectedObject.yUnit == "%") {state.selectedObject.y = ((state.selectedObject.y)/state.canvas.height)*100}}}>
                                    <option value="px">px</option>
                                    <option value="%">%</option>
                                </select>
                            </div>
                        </div>
                        <div class="options__input">
                            <div class="input__container">
                                <span>Text Anchor:</span>
                                <select name="" class="wide" bind:value={state.selectedObject.textAnchor}>
                                    <option value="start">Start</option>
                                    <option value="middle">Middle</option>
                                    <option value="end">End</option>
                                </select>
                            </div>
                            <div class="input__container">
                                <span>Baseline:</span>
                                <select name="" class="wide" bind:value={state.selectedObject.dominantBaseline}>
                                    <option value="hanging">Top</option>
                                    <option value="middle">Center</option>
                                    <option value="auto">Bottom</option>
                                </select>
                            </div>
                        </div>
                        <div class="options__input">
                            <div class="input__container">
                                <span>Font size:</span>
                                <input type="number" style="width: 35px" bind:value={state.selectedObject.fontSize} min="0" on:change={() =>{if (state.selectedObject.fontSize == null) state.selectedObject.fontSize = 1}}>
                            </div>
                            <div class="input__container">
                                <span>Font Weight:</span>
                                <input type="number" style="width: 35px" bind:value={state.selectedObject.fontWeight} min="100" on:change={() =>{if (state.selectedObject.fontWeight == null) state.selectedObject.fontWeight = 100}}>
                            </div>
                        </div>
                        <div class="options__input">
                            <div class="input__container">
                                <span>Font Color:</span>
                                <input type="text" style="width: 100px" bind:value={state.selectedObject.color} on:change={() =>{if (state.selectedObject.color == null) state.selectedObject.color = "#000"}}>
                            </div>
                        </div>
                        <div class="options__input">
                            <div class="input__container">
                                <span>Text Decoration:</span>
                                <select name="" class="wide" bind:value={state.selectedObject.textDecoration}>
                                    <option value="none">None</option>
                                    <option value="underline">Underline</option>
                                    <option value="line-through">Line-through</option>
                                </select>
                            </div>
                        </div>
                        <div class="options__input" style="margin:0">
                            <div class="input__container" style="display:flex">
                                <span>Italic:&MediumSpace;</span>
                                <label class="container">
                                    <input type="checkbox" bind:checked={state.selectedObject.italic} style="display:none">
                                    <span class="checkmark"></span>
                                </label>
                            </div>
                        </div>
                        <button class="option__btn" style="border-right: 1px solid #222" on:click={_delete}>Delete</button>
                        <button class="option__btn" style="border-right: 1px solid #222" on:click={state.textObjects.duplicateObject}>Duplicate</button>
                        <div class="text__area">
                            <div class="input__container" style="display:grid;grid-template-columns:100px 1fr;">
                                <span>Text Content:</span>
                                <input type="text" style="text-align:start; width:auto" bind:value={state.selectedObject.text} min="0" on:change={() =>{if (state.selectedObject.text == "") state.selectedObject.text = "Empty"; updateTitle()}}>
                            </div>
                        </div>
                    </div>
                {/key}
            {/if}
        {:else}
            <div class="options">
                <button class="option__btn" style="border-right: 1px solid #222" on:click={()=>{state.exportMode = false;transPos={x:0,y:0};panTrans="translate3D(0,0,0)";scale = 1; document.getElementById("scaler").style.transform = ""}}>Exit Export</button>
                <button class="option__btn" style="border-right: 1px solid #222" on:click={copy}>Copy Code to Clipboard</button>
                <button class="option__btn" style="border-right: 1px solid #222" on:click={()=>{downloadModal = true}}>Download</button>
                <button class="option__btn" style="border-right: 1px solid #222" on:click={()=>{updatePreview("d")}}>Preview Desktop</button>
                <div class="options__input">
                    <div class="input__container">
                        <span>Mobile Device Preview:</span>
                        <select name="" class="wide" bind:value={selectedMobile} on:change={()=>{updatePreview("m")}}>
                            <option value="none">None</option>
                            <option value="iphonese">iPhone SE</option>
                            <option value="iphonexr">iPhone XR</option>
                            <option value="iphone12pro">iPhone 12 Pro</option>
                            <option value="pixel5">Pixel 5</option>
                            <option value="samsunggalaxys8">Samsung Galaxy S8+</option>
                            <option value="samsunggalaxys20ultra">Samsung Galaxy S20 Ultra</option>
                        </select>
                    </div>
                </div>
                <div class="options__input">
                    <div class="input__container">
                        <span>Tablet Device Preview:</span>
                        <select name="" class="wide" bind:value={selectedTablet} on:change={()=>{updatePreview("t")}}>
                            <option selected value="none">None</option>
                            <option value="ipadair">iPad Air</option>
                            <option value="ipadmini">iPad Mini</option>
                            <option value="surfacepro">Surface Pro 7</option>
                        </select>
                    </div>
                </div>
                <div class="options__input">
                    <div class="input__container">
                        <span>Preview Height:</span>
                        <input type="number" bind:value={previewHeight} min="0" on:change={()=>{updatePreview("r")}}>
                    </div>
                    <div class="input__container">
                        <span>Preview Width:</span>
                        <input type="number" bind:value={previewWidth} min="0" on:change={()=>{updatePreview("r")}}>
                    </div>
                </div>
            </div>
        {/if}
    </div>
    {#if state.exportMode}
    <div class="config__menu">
        {#if wrapped == false}
        <button class="modal__btn" on:click={wrapExport}>Wrap in a Container</button>
        {:else}
        <button class="modal__btn" on:click={wrapExport}>Unwrap</button>
        <div class="input__container">
            <span>Container Height:</span>
            <input type="number" bind:value={containerHeight} min="10" on:change={_export}>
        </div>
        {/if}
        <div class="input__container">
            <span>Aria Title:</span>
            <input style="width:100%" type="text" bind:value={ariaTitle} on:change={_export}>
        </div>
        <div class="input__container">
            <span>Aria Description:</span>
            <textarea type="text" bind:value={ariaDesc} on:change={_export}/>
        </div>
    </div>
    {/if}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="canvas__container" on:click={objClick} on:wheel={canvZoom} on:mousedown={pan}>
        <div id="scaler">
            {#if !state.exportMode}
                {#key state.loaded == true}
                    <svg class="canvas" id="cnvs" height={state.canvas.getHeight()} width={state.canvas.getWidth()} viewbox={state.canvas.getViewbox()} style="background-color: {state.canvas.background}">
                        {#if state.textObjects.objectList.length != 0}
                            {#each state.textObjects.objectList as obj}
                                <text class="text__object" id={obj.id} text-decoration={obj.textDecoration} font-style={obj.italic ? "italic" : "none"} fill={obj.color} font-weight={obj.fontWeight} dominant-baseline={obj.dominantBaseline} text-anchor={obj.textAnchor} x={obj.x+obj.xUnit} y={obj.y+obj.yUnit} font-size={obj.fontSize+"px"}>{obj.text}</text>
                            {/each}
                        {/if}
                    </svg>
                {/key}
            {:else}
                <div class="preview__canvas" style={`height:${previewHeight}px;width:${previewWidth}px;`} contenteditable="false" bind:innerHTML={exportedCode}></div>
            {/if}
        </div>
        <div class="trans_btns">
            {#if (scale != 1)}
            <button class="zoom__btn" on:click={()=>{scale=1; let c = document.getElementById("scaler") ;c.style.transform = `scale(${scale})`}}>Reset Zoom</button>
            <span style="color:#0dd" class="zoom-text">Current Zoom: {(scale * 100).toFixed(2)}%</span>
            {/if}
            {#if (transPos.x != 0 || transPos.y != 0)}
            <button class="recenter__btn" on:click={()=>{transPos={x:0,y:0};panTrans="translate3D(0,0,0)"; let c = state.exportMode ? document.querySelector(".preview__canvas") : document.getElementById("cnvs"); c.style.transform = `${`translate3D(${transPos.x}px,${transPos.y }px,0)`}`;}}>Center</button>
            {/if}
        </div>
    </div>
</div>

<!-- 
    SVG Text Builder

    This build tool allows you to easily create a typographical svg component.
    
    Features:
     Templates for common use cases, such as promo banners or badges
     Canvas is resizable, which optionally translates into the viewbox of the svg
     Select a text object by clicking on it
     Edit properties of that object using the menu
     Move text objects by dragging
     Panning & Zooming
     Autosaves last 10 projects
     Saves menu for creating unlimited manual saves
     Edit history (Undo, Redo), ex. CTRL + Z, CTRL + Y. Saves last 999 Edits
     Export features:
        Download as svg, html, png, jpeg, webp.
        Preview the SVG on multiple different devices
        Copy the HTML for the SVG to clipboard

    Planned Features:
     Select by dragging  
     Select multiple objects as a group to move together (Not sure if I'll let them change multiple properties at the same time, but you can move the group)
     Import non-text SVGs in a <g> so the whole thing can be moved at once
     Import/Export .html or .svg files
     Add in accesibility control for exported SVGs
    
    Stretch Features:
     Allow people to save creations as templates for others to browse through and load into the tool
     
    Known Bugs:
     Sometimes the object hover border box breaks and sits at the top left with a small size
 -->