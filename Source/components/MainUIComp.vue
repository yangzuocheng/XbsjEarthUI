<template>
    <div style="width:100%;height:100%">
        <!-- 导航栏部分 -->
        <MainBarControl ref="mainBarControl"></MainBarControl>
        <div class="xbsjcesium" ref="xbsjcesium">
            <div ref="cesiumContainer" style="width: 100%; height: 100%"></div>
            <!-- 视口分屏 -->
            <ViewportLine></ViewportLine>
        </div>
        <StatusBarControl ref="statusBarControl"></StatusBarControl>
        <!-- <NavigatorControl ref="navigator"></NavigatorControl> -->
        <TimelineBarControl ref="timelineBarControl"></TimelineBarControl>
        <component
            v-for="(tool,index) in tools"
            :is="tool.component"
            :ref="tool.ref"
            :_toolIndex="index"
            :key="_getToolID(tool)"
            :getBind="tool.item"
            :nextczm="tool.nextczm"
        ></component>
        <ContextMenu ref="contextMenu"></ContextMenu>
        <!-- 只有一个 colorpicker -->
        <GlobalColorPicker ref="colorPicker"></GlobalColorPicker>
        <InformationBox
            v-for="(info,index) in infos"
            :_toolIndex="index"
            :info="info.info"
            :type="info.type"
            :key="info.key"
            :style="{top:(140+index*30)+'px'}"
        ></InformationBox>
        <!-- 只有一个 Modal 对话框 方便调用 -->
        <Modal :visible="modal" @cancel="modalCancel" @confirm="modalConfirm">{{confirmInfo}}</Modal>
        <Window
            :footervisible="true"
            @cancel="loadGeoJSONShow=false"
            @ok="confirmLoadGeoJson"
            v-show="loadGeoJSONShow"
            :width="300"
            :minWidth="200"
            :height="500"
            :left="500"
            :top="138"
            :title="lang.title"
        >
            <div class="group">
                <ul>
                    <li v-for="(p,index) in types" @click="selectType(index, p)" :key="index" :class="{active:categoryIndex==index}">
                        <label class="liname">{{p.name}}</label>
                        <label class="livalue">{{p.typeName}}</label>
                    </li>
                </ul>
            </div>
        </Window>
    </div>
</template>

<script>
import MainBarControl from './controls/MainBarControl'
import TimelineBarControl from './controls/TimelineControl'
import StatusBarControl from './controls/StatusBarControl.vue'
import NavigatorControl from './controls/NavigatorControl.vue'
import ViewportLine from './controls/ViewportLine'

import SceneTreeTool from './tools/SceneTreeTool'
import ImageryLab from './tools/ImageryServices/ImageryLab.vue'
import ImageryCloud from './tools/ImageryServices/ImageryCloud.vue'
import ImageryOnline from './tools/ImageryServices/ImageryOnline.vue'
import ImageryWMTS from './tools/ImageryServices/ImageryWMTS.vue'
import ImageryWMS from './tools/ImageryServices/ImageryWMS.vue'
import ImageryArcGis from './tools/ImageryServices/ImageryArcGis.vue'
import DefaultAccessToken from './tools/ImageryServices/defaultAccessToken.vue'

import ModelLab from './tools/ModelServices/ModelLab.vue'
import ModelCloud from './tools/ModelServices/ModelCloud.vue'
import ModelOnline from './tools/ModelServices/ModelOnline.vue'
import TerrainLab from './tools/TerrainServices/TerrainLab.vue'
import TerrainCloud from './tools/TerrainServices/TerrainCloud.vue'
import TerrainOnline from './tools/TerrainServices/TerrainOnline.vue'
import CameraViewManager from './tools/CameraViewManager'
import CutFillComputing from './tools/CutFillComputing'
import FeatureProperty from './tools/FeatureProperty'

import ContextMenu from './common/ContextMenu'

import FlattenningTool from './viztools/FlattenningTool'
import ClippingPlaneTool from './viztools/ClippingPlaneTool'
import WaterTool from './viztools/WaterTool'
import PinTool from './viztools/PinTool'
import PinDiv from './viztools/PinDiv'
import PinPictureTool from './viztools/PinPictureTool'
import GroundImageTool from './viztools/GroundImageTool'
import PinDivTool from './viztools/PinDivTool'
import PathTool from './viztools/PathTool'
import PolygonTool from './viztools/PolygonTool'
import ClassificationPolygonTool from './viztools/ClassificationPolygonTool'
import ForestTool from './viztools/ForestTool'
import ModelTool from './viztools/ModelTool'
import PolylineTool from './viztools/PolylineTool'
import GeoCurveArrow from './viztools/GeoCurveArrow'
import GeoFlatArrow from './viztools/GeoFlatArrow'
import GeoParticleSystem from './viztools/GeoParticleSystem'
import GeoParticleSystemFireWork from './viztools/GeoParticleSystemFireWork'
import GeoParticleSystemTails from './viztools/GeoParticleSystemTails'
import GeoSmoothPolygon from './viztools/GeoSmoothPolygon'
import GeoCurve from './viztools/GeoCurve'
import GeoDoubleArrow from './viztools/GeoDoubleArrow'
import GeoCircle from './viztools/GeoCircle'
import GeoRectangle from './viztools/GeoRectangle'
import GeoCurveFlag from './viztools/GeoCurveFlag'
import GeoRightAngleFlag from './viztools/GeoRightAngleFlag'
import GeoPolygon from './viztools/GeoPolygon'
import GeoSectorSearch from './viztools/GeoSectorSearch'
import GeoPolylineArrow from './viztools/GeoPolylineArrow'
import GeoPolyline from './viztools/GeoPolyline'
import GeoArc from './viztools/GeoArc'
import GeoBezier2 from './viztools/GeoBezier2'
import GeoBezier3 from './viztools/GeoBezier3'
import GeoParallelSearch from './viztools/GeoParallelSearch'
import GeoTriFlag from './viztools/GeoTriFlag'
import GeoSector from './viztools/GeoSector'
import ScanlineTool from './viztools/ScanlineTool'
import CustomPrimitiveTool from './viztools/CustomPrimitiveTool'
import CustomGroundRectangleTool from './viztools/CustomGroundRectangleTool'
import RoadTool from './viztools/RoadTool'
import WallTool from './viztools/WallTool'
import SurfaceTool from './viztools/SurfaceTool'
import CutSurfaceTool from './viztools/CutSurfaceTool'
import TubeTool from './viztools/TubeTool'
import AddPoint from './viztools/ForestTool/AddPoint'
import ForestLab from './viztools/ForestTool/ForestLab'
import LinearPanel from './viztools/ForestTool/LinearPanel'
import FacePlate from './viztools/ForestTool/FacePlate'

import CamerVideoTool from './viztools/CamerVideoTool'
import ViewshedTool from './viztools/ViewshedTool'

import PropertyWindow from './properties/PropertyWindow'
import CameraViewPrp from './properties/CameraViewPrp'
import ProfileAnalysis from './properties/ProfileAnalysis'
import TilesetStyleEditor from './properties/TilesetStyleEditor'
import TilesetExpansionEditor from './properties/TilesetExpansionEditor'
import ShareEditor from './properties/ShareEditor'

import GlobalColorPicker from './common/GlobalColorPicker'
import InformationBox from './utils/InformationBox'

import ModelTreeTool from './tools/ModelTreeTool'
import EntityMoreTool from './tools/EntityMoreTool'
import CustomSymbol from './tools/SymbolTool/CustomSymbol'
import LabSymbol from './tools/SymbolTool/LabSymbol'
import OnlineSymbol from './tools/SymbolTool/OnlineSymbol'
import TilesTest from './tools/TilesTest'

import GeoPolygonImage from './viztools/GeoPolygonImage'
import CesiumDataSource from './viztools/CesiumDataSource'
import WMTSHistory from './viztools/WMTSHistory'
export default {
	components: {
		StatusBarControl,
		NavigatorControl,
		MainBarControl,
		TimelineBarControl,
		SceneTreeTool,
		ContextMenu,
		ViewportLine,
		CameraViewManager,
		CutFillComputing,
		FeatureProperty,

		FlattenningTool,
		ClippingPlaneTool,
		WaterTool,
		PinTool,
		PinDiv,
		PinPictureTool,
		GroundImageTool,
		PinDivTool,
		PathTool,
		PolygonTool,
		ClassificationPolygonTool,
		ForestTool,
		ModelTool,
		PolylineTool,
		GeoCurveArrow,
		GeoCurve,
		GeoDoubleArrow,
		GeoCircle,
		GeoRectangle,
		GeoCurveFlag,
		GeoRightAngleFlag,
		GeoPolygon,
		GeoSectorSearch,
		GeoPolylineArrow,
		GeoPolyline,
		GeoArc,
		GeoBezier2,
		GeoBezier3,
		GeoParallelSearch,
		GeoTriFlag,
		GeoSector,
		GeoFlatArrow,
		GeoParticleSystem,
		GeoParticleSystemFireWork,
		GeoParticleSystemTails,
		GeoSmoothPolygon,
		RoadTool,
		WallTool,
		SurfaceTool,
		CutSurfaceTool,
		ScanlineTool,
		CustomPrimitiveTool,
		CustomGroundRectangleTool,
		TubeTool,
		CamerVideoTool,
		ViewshedTool,
		WMTSHistory,

		ImageryLab,
		ImageryCloud,
		ImageryOnline,
		ImageryWMTS,
		ImageryWMS,
		ImageryArcGis,
		DefaultAccessToken,
		ModelLab,
		ModelCloud,
		ModelOnline,
		TerrainLab,
		TerrainCloud,
		TerrainOnline,
		PropertyWindow,
		CameraViewPrp,
		ProfileAnalysis,
		AddPoint,
		ForestLab,
		LinearPanel,
		FacePlate,
		TilesetStyleEditor,
		TilesetExpansionEditor,
		ShareEditor,
		GlobalColorPicker,
		InformationBox,
		ModelTreeTool,
		EntityMoreTool,
		CustomSymbol,
		LabSymbol,
		OnlineSymbol,
		TilesTest,
		GeoPolygonImage,
		CesiumDataSource
	},
	data: function() {
		return {
			modal: false,
			confirmInfo: '',
			registerComponents: {
				// czmObject的类型和组件名称的关联表，如果为绑定，则使用TreeCommon vtxf 190629
				FlattenedPolygonCollection: 'FlattenningTool',
				CameraVideo: 'CamerVideoTool',
				Viewshed: 'ViewshedTool',
				ClippingPlane: 'ClippingPlaneTool',
				Water: 'WaterTool',
				Pin: 'PinTool',
				GeoPin: 'PinDiv',
				PinDivTool: 'PinDivTool',
				// Pin: "PinPictureTool",
				Path: 'PathTool',
				Polygon: 'PolygonTool',
				ClassificationPolygon: 'ClassificationPolygonTool',
				Forest: 'ForestTool',
				Scanline: 'ScanlineTool',
				CustomPrimitive: 'CustomPrimitiveTool',
				CustomGroundRectangle: 'CustomGroundRectangleTool',
				Road: 'RoadTool',
				Wall: 'WallTool',
				Surface: 'SurfaceTool',
				CutSurface: 'CutSurfaceTool',
				CustomPrimitiveExt_Tube: 'TubeTool',
				Model: 'ModelTool',
				Polyline: 'PolylineTool',
				GeoCurveArrow: 'GeoCurveArrow',
				GeoCurve: 'GeoCurve',
				GeoSectorSearch: 'GeoSectorSearch',
				GeoPolylineArrow: 'GeoPolylineArrow',
				GeoPolyline: 'GeoPolyline',
				GeoArc: 'GeoArc',
				GeoBezier2: 'GeoBezier2',
				GeoBezier3: 'GeoBezier3',
				GeoParallelSearch: 'GeoParallelSearch',
				GeoTriFlag: 'GeoTriFlag',
				GeoDoubleArrow: 'GeoDoubleArrow',
				GeoCircle: 'GeoCircle',
				GeoArrow: 'GeoFlatArrow',
				ParticleSystem: 'GeoParticleSystem',
				ParticleSystemFireWork: 'GeoParticleSystemFireWork',
				ParticleSystemTails: 'GeoParticleSystemTails',
				GeoSmoothPolygon: 'GeoSmoothPolygon',
				GeoRectangle: 'GeoRectangle',
				GeoCurveFlag: 'GeoCurveFlag',
				GeoRightAngleFlag: 'GeoRightAngleFlag',
				GeoPolygon: 'GeoPolygon',
				GeoSector: 'GeoSector',
				['CameraView.View']: 'CameraViewPrp',
				ProfileAnalysis: 'ProfileAnalysis',
				GroundImage: 'GroundImageTool',
				// GeoPin: "PinDivTool",
				TilesTest: 'TilesTest',
				CustomPrimitiveExt_Image: 'GeoPolygonImage',
				XbsjGeoJSON: 'CesiumDataSource',
				XbsjKML: 'CesiumDataSource',
				XbsjCzml: 'CesiumDataSource'
			},
			tools: [
				{
					component: 'SceneTreeTool',
					ref: 'sceneTreeTool'
				},
				{
					component: 'ImageryLab',
					ref: 'imageryLab'
				},
				{
					component: 'ImageryCloud',
					ref: 'ImageryCloud'
				},
				{
					component: 'ImageryOnline',
					ref: 'imageryOnline'
				},
				{
					component: 'ImageryWMTS',
					ref: 'imageryWMTS'
				},
				{
					component: 'ImageryWMS',
					ref: 'imageryWMS'
				},
				{
					component: 'ImageryArcGis',
					ref: 'imageryArcGis'
				},
				{
					component: 'DefaultAccessToken',
					ref: 'defaultAccessToken'
				},
				{
					component: 'ModelLab',
					ref: 'modelLab'
				},
				{
					component: 'ModelCloud',
					ref: 'modelCloud'
				},
				{
					component: 'ModelOnline',
					ref: 'modelOnline'
				},
				{
					component: 'TerrainLab',
					ref: 'terrainLab'
				},
				{
					component: 'TerrainCloud',
					ref: 'terrainCloud'
				},
				{
					component: 'TerrainOnline',
					ref: 'terrainOnline'
				},
				{
					component: 'CameraViewManager',
					ref: 'cameraViewManager'
				},
				{
					component: 'CutFillComputing',
					ref: 'cutFillComputing'
				},
				{
					component: 'FeatureProperty',
					ref: 'featureProperty'
				},
				{
					component: 'ModelTreeTool',
					ref: 'modelTreeTool'
				},
				{
					component: 'EntityMoreTool',
					ref: 'entitymoreTool'
				},
				{
					component: 'CustomSymbol',
					ref: 'customSymbol'
				},
				{
					component: 'LabSymbol',
					ref: 'labSymbol'
				},
				{
					component: 'OnlineSymbol',
					ref: 'onlineSymbol'
				},
				{
					component: 'TilesTest',
					ref: 'tilesTest'
				}
			],
			infos: [],
			jsontext: '',
			loadGeoJSONShow: false,
			types: null,
			categoryIndex: 0,
			selectedType: null
		}
	},
	mounted() {
		let xbsjcesium = this.$refs.xbsjcesium
		let that = this

		function handleDragOver(e) {
			e.stopPropagation()
			e.preventDefault()
		}

		function handleFileSelect(e) {
			// e.stopPropagation();
			e.preventDefault()
			let item = e.dataTransfer

			var files = []
			;[].forEach.call(
				e.dataTransfer.files,
				function(file) {
					files.push(file)
				},
				false
			)

			for (var f of files) {
				var reader = new FileReader()
				reader.readAsText(f)
				//读取文件的内容
				reader.onload = function() {
					that.jsontext = JSON.parse(this.result)
					that.analysisJson()
				}
			}
		}

		xbsjcesium.addEventListener('dragover', handleDragOver, false)
		xbsjcesium.addEventListener('drop', handleFileSelect, false)
		;(this.polylineTypes = [
			{
				name: '线',
				typeName: 'Plots.GeoPolyline',
				getObj: function(earth) {
					return new XE.Obj.Plots.GeoPolyline(earth)
				}
			}
			// {
			//   name: "管道",
			//   typeName: "CustomPrimitiveExt.Tube",
			//   getObj: function(earth) {
			//     var tube = new XE.Obj.CustomPrimitiveExt.Tube(earth);
			//     tube.imageUrl = "../../assets/ht/meteor_01.png";
			//     tube.radius = 0.5;
			//     tube.speed = [0.2, 0.2];
			//     return tube;
			//   }
			// }
		]),
			(this.polygonTypes = [
				{
					name: '面',
					typeName: 'Plots.GeoPolygon',
					getObj: function(earth) {
						return new XE.Obj.Plots.GeoPolygon(earth)
					}
				}
			])
	},
	computed: {
		type() {
			return this.viewporttype
		}
	},
	methods: {
		confirmLoadGeoJson() {
			if (this.jsontext.type != '') {
				const g0 = new XE.SceneTree.Group(this.$root.$earth)
				g0.title = '图形组合文件夹'
				g0.isSelected = true
				const xbsjSceneTree = this.$root.$earth.sceneTree
				xbsjSceneTree.root.children.push(g0)
			}
			let arr
			if (this.jsontext.features && this.jsontext.features.length > 0) {
				arr = this.jsontext.features
				var len = arr.length
				if (arr.length > 100) {
					len = 100
					this.$root.$earthUI.promptInfo('只创建前100条！', 'warning')
				}
				for (let j = 0; j < len; j++) {
					if (arr[j].geometry.type.toLowerCase() === 'linestring') {
						var polylin = this.selectedType.getObj(this.$root.$earth)
						if (arr[j].properties && arr[j].properties.name) {
							polylin.name = arr[j].properties && arr[j].properties.name
						}
						var positionarr = arr[j].geometry.coordinates
						for (let k = 0; k < positionarr.length; k++) {
							positionarr[k][0] = (Math.PI * positionarr[k][0]) / 180.0
							positionarr[k][1] = (Math.PI * positionarr[k][1]) / 180.0
							if (positionarr[k].length > 2) {
								positionarr[k][2] = positionarr[k][2]
								polylin.ground = false
							} else {
								positionarr[k][2] = 0
							}
						}
						// π/180×角度
						polylin.positions = positionarr
						var selected = this.$root.$earth.sceneTree.currentSelectedNode
						const obj = new XE.SceneTree.Leaf(polylin)
						selected.children.push(obj)
					} else if (arr[j].geometry.type.toLowerCase() === 'polygon') {
						var polygon = this.selectedType.getObj(this.$root.$earth)
						if (arr[j].properties && arr[j].properties.name) {
							polygon.name = arr[j].properties && arr[j].properties.name
						}
						var positionarr = arr[j].geometry.coordinates[0]
						for (let k = 0; k < positionarr.length; k++) {
							positionarr[k][0] = (Math.PI * positionarr[k][0]) / 180.0
							positionarr[k][1] = (Math.PI * positionarr[k][1]) / 180.0
							if (positionarr[k].length > 2) {
								positionarr[k][2] = positionarr[k][2]
								polygon.ground = false
							} else {
								positionarr[k][2] = 0
							}
						}
						// π/180×角度
						polygon.positions = positionarr
						var selected = this.$root.$earth.sceneTree.currentSelectedNode
						const obj = new XE.SceneTree.Leaf(polygon)
						selected.children.push(obj)
					} else if (
						arr[j].geometry.type.toLowerCase() === 'multilinestring' ||
						arr[j].geometry.type.toLowerCase() === 'multipolygon'
					) {
						var multiLineString = arr[j].geometry.coordinates
						for (let single = 0; single < multiLineString.length; single++) {
							var polylin = this.selectedType.getObj(this.$root.$earth)
							if (arr[j].properties && arr[j].properties.name) {
								polylin.name = arr[j].properties && arr[j].properties.name
							}
							positionarr = multiLineString[single]
							for (let k = 0; k < positionarr.length; k++) {
								positionarr[k][0] = (Math.PI * positionarr[k][0]) / 180.0
								positionarr[k][1] = (Math.PI * positionarr[k][1]) / 180.0
								if (positionarr[k].length > 2) {
									positionarr[k][2] = positionarr[k][2]
									polylin.ground = false
								} else {
									positionarr[k][2] = 0
								}
							}

							// π/180×角度
							polylin.positions = positionarr
							var selected = this.$root.$earth.sceneTree.currentSelectedNode
							const obj = new XE.SceneTree.Leaf(polylin)
							selected.children.push(obj)
						}
					}
				}
			}
			this.loadGeoJSONShow = false
		},
		analysisJson() {
			if (this.jsontext.sceneTree) {
				let self = this
				this.confirm(
					'是否替换当前场景？',
					() => {
						self.$root.$earth.xbsjFromJSON(this.jsontext)
						// self.$root.$earth.sceneTree.root.children.push(
						//   this.jsontext.sceneTree.root
						// );
					}
					// () => {
					//   self.$root.$earth.sceneTree.root.children.push(
					//     this.jsontext.sceneTree.root
					//   );
					// }
				)
			} else if (this.jsontext.czmObject) {
				this.$root.$earth.sceneTree.root.children.push(this.jsontext)
			} else if (this.jsontext.xbsjType) {
				var czmObject = {}
				czmObject.czmObject = this.jsontext
				this.$root.$earth.sceneTree.root.children.push(czmObject)
			} else if (this.jsontext.children && this.jsontext.children.length >= 0) {
				this.$root.$earth.sceneTree.root.children.push(this.jsontext)
			} else {
				let arr
				if (this.jsontext.features && this.jsontext.features.length > 0) {
					arr = this.jsontext.features
				} else if (this.jsontext.geometries && this.jsontext.geometries.length > 0) {
					arr = []
					for (var i = 0; i < this.jsontext.geometries.length; i++) {
						arr.push({ geometry: this.jsontext.geometries[i] })
					}
					this.jsontext.features = arr
				}
				if (arr && arr.length > 0) {
					if (arr.length > 0) {
						if (arr[0].geometry.type.toLowerCase() === 'polygon' || arr[0].geometry.type.toLowerCase() === 'multipolygon') {
							this.types = this.polygonTypes
							this.loadGeoJSONShow = true
							this.selectedType = this.types[0]
						} else if (
							arr[0].geometry.type.toLowerCase() === 'linestring' ||
							arr[0].geometry.type.toLowerCase() === 'multilinestring'
						) {
							this.types = this.polylineTypes
							this.loadGeoJSONShow = true
							this.selectedType = this.types[0]
						}
					}
				}
			}
		},
		selectType(index, item) {
			this.categoryIndex = index
			this.selectedType = item
		},
		_getToolID(tool) {
			if (!tool.guid) {
				tool.guid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
					var r = (Math.random() * 16) | 0
					var v = c === 'x' ? r : (r & 0x3) | 0x8
					return v.toString(16)
				})
			}
			return tool.guid
		},
		confirm(info, ok, cancel) {
			this.confirmInfo = info
			this.modal = true
			this._ok = ok
			this._cancel = cancel
		},
		modalCancel() {
			this.modal = false
			if (typeof this._cancel == 'function') {
				this._cancel()
			}
		},
		modalConfirm() {
			this.modal = false
			if (typeof this._ok == 'function') {
				this._ok()
			}
		},

		//显示对象的属性窗口
		showPropertyWindow(czmObject, options) {
			//一个对象可以弹出若干种不同类型的属性窗口,判定是哪种component

			//用于判断是否弹出属性面板--mrq
			// try {
			//   if (!((options && options.jsonSchema) || czmObject._jsonSchema)) {
			//     return;
			//   }
			// } catch (error) {
			//   return
			// }

			//默认是统一类型的属性窗口
			var component
			// console.log(czmObject.ctrtype);
			//如果有外部配置
			if (options && (typeof options.component == 'string' || typeof options.component == 'object')) {
				component = options.component
			}

			//如果有默认映射
			else if (czmObject.xbsjType) {
				if (czmObject.ctrtype) {
					var c = this.registerComponents[czmObject.ctrtype]
					if (c) {
						component = c
					}
				} else {
					var c = this.registerComponents[czmObject.xbsjType]
					if (c) {
						component = c
					}
				}
			}

			//如果没有comp，那么判断是否要弹出默认属性面板
			if (!component) {
				try {
					if ((options && options.jsonSchema) || (czmObject && czmObject._jsonSchema)) {
						component = 'PropertyWindow'
					} else {
						console.log('no configed component')
						return
					}
				} catch (ex) {
					console.log('no configed component' + ex)
					return
				}
			}
			//判定是否有属性窗口
			const index = this.tools.findIndex(e => {
				//类型不一致，直接返回
				if (e.component != component) return false
				//绑定的对象一致的
				return e.item && e.item() === czmObject
			})
			//，如果有放到最前
			if (index >= 0) {
				this._topWindow(index)
				return
			}

			var guid = czmObject.guid || Cesium.createGuid()
			if (typeof component == 'string') {
				guid += '_' + component
			}
			//新建窗口
			this.tools.push({
				component: component,
				ref: guid,
				guid: guid,
				item: () => {
					return czmObject
				},
				//mrq添加
				nextczm: options && options.jsonSchema
			})
		},
		_topWindow(index) {
			if (index < 0 && index == this.tools.length - 1) return

			const tool = this.tools[index]
			this.tools.splice(index, 1)
			this.tools.push(tool)
		},
		topWindow(window) {
			const index = window.$parent.$attrs._toolIndex
			this._topWindow(index)
		},
		destroyTool(tool) {
			const index = tool.$attrs._toolIndex
			if (index !== -1) {
				//const tool = this.tools[index];
				this.tools.splice(index, 1)
			}
		},

		promptInfo(info, type) {
			var _info = {
				info,
				type,
				key: Cesium.createGuid()
			}
			this.infos.push(_info)
			//开始计时，3s之后移除这个info
			setTimeout(() => {
				const index = this.infos.findIndex(i => {
					return i.key === _info.key
				})
				if (index >= 0) {
					this.infos.splice(index, 1)
				}
			}, 3000)
		},

		// 获取拖拽对象
		getCzmObjectFromDrag(dataTransfer) {
			for (let i = 0; i < dataTransfer.types.length; i++) {
				var t = dataTransfer.types[i]
				if (!t) continue
				if (t.startsWith('_czmobj_')) {
					let guid = t.substring(8)

					return this.$root.$earth.getObject(guid)
				}
			}
			return undefined
		},

		// 获取拖拽对象位置
		getCzmObjectPositionFromDrag(dragczmObj, czmObj) {
			if (dragczmObj._polyline !== undefined) {
				czmObj.positions = [...dragczmObj._polyline.positions]
			} else if (dragczmObj._polygon !== undefined) {
				var polygonPositions = [],
					polygonPositions2 = []
				// 将数组分割成每两个一组
				for (var i = 0; i < dragczmObj._polygon.positions.length; i += 2) {
					polygonPositions.push(dragczmObj._polygon.positions.slice(i, i + 2))
				}
				polygonPositions2 = polygonPositions.map(e => {
					e[2] = dragczmObj._polygon.height
					return e
				})
				czmObj.positions = [...polygonPositions2]
			} else {
				czmObj.positions = [...dragczmObj.positions]
			}
		},

		// 获取拖拽对象位置
		getCzmObjectPositionFromDrags(dragczmObj, czmObj) {
			if (dragczmObj._polyline !== undefined) {
				var polylinePositions = [],
					polylinePositions2 = []
				// 将数组分割成每两个一组
				for (var i = 0; i < dragczmObj._polyline.positions.length; i++) {
					polylinePositions.push(dragczmObj._polyline.positions[i].slice(0, 2))
				}
				polylinePositions.forEach(element => {
					polylinePositions2.push(element[0])
					polylinePositions2.push(element[1])
				})
				czmObj.positions = [...polylinePositions2]
				czmObj.height = dragczmObj._polyline.positions[0][2]
			} else if (dragczmObj._polygon !== undefined) {
				czmObj.positions = [...dragczmObj._polygon.positions]
				czmObj.height = dragczmObj._polygon.height
			} else {
				czmObj.positions = [...dragczmObj.positions]
			}
		}
	}
}
</script>

<style>
.xbsjcesium {
	width: 100%;
	height: calc(100% - 137px);
	position: relative;
}
.active {
	background: rgba(200, 200, 200, 0.5);
}
</style>
