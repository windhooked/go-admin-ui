<template>
  <div class="right-board">
    <el-tabs v-model="currentTab" class="center-tabs">
      <el-tab-pane label="Component attributes" name="field" />
      <el-tab-pane label="Form attributes" name="form" />
    </el-tabs>
    <div class="field-box">
      <a class="document-link" target="_blank" :href="documentLink" title="View component documentation">
        <i class="el-icon-link" />
      </a>
      <el-scrollbar class="right-scrollbar">
        <!-- Component properties -->
        <el-form v-show="currentTab==='field' && showField" size="small" label-width="90px">
          <el-form-item v-if="activeData.changeTag" label="component type">
            <el-select
              v-model="activeData.tagIcon"
              placeholder="Please select component type"
              :style="{width: '100%'}"
              @change="tagChange"
            >
              <el-option-group v-for="group in tagList" :key="group.label" :label="group.label">
                <el-option
                  v-for="item in group.options"
                  :key="item.label"
                  :label="item.label"
                  :value="item.tagIcon"
                >
                  <svg-icon class="node-icon" :icon-class="item.tagIcon" />
                  <span> {{ item.label }}</span>
                </el-option>
              </el-option-group>
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.vModel!==undefined" label="activeData vmodel">
            <el-input v-model="activeData.vModel" placeholder="Please enter the field name (v-model)" />
          </el-form-item>
          <el-form-item v-if="activeData.componentName!==undefined" label="activeData Comonent Name">
            {{ activeData.componentName }}
          </el-form-item>
          <el-form-item v-if="activeData.label!==undefined" label="Label">
            <el-input v-model="activeData.label" placeholder="Please enter a title" />
          </el-form-item>
          <el-form-item v-if="activeData.placeholder!==undefined" label="placeholder tips">
            <el-input v-model="activeData.placeholder" placeholder="Please enter a placeholder hint" />
          </el-form-item>
          <el-form-item v-if="activeData['start-placeholder']!==undefined" label="start placeholder">
            <el-input v-model="activeData['start-placeholder']" placeholder="Please enter a placeholder hint" />
          </el-form-item>
          <el-form-item v-if="activeData['end-placeholder']!==undefined" label="end placeholder">
            <el-input v-model="activeData['end-placeholder']" placeholder="Please enter a placeholder hint" />
          </el-form-item>
          <el-form-item v-if="activeData.span!==undefined" label="form grid">
            <el-slider v-model="activeData.span" :max="24" :min="1" :marks="{12:''}" @change="spanChange" />
          </el-form-item>
          <el-form-item v-if="activeData.layout==='rowFormItem'" label="Grid spacing">
            <el-input-number v-model="activeData.gutter" :min="0" placeholder="grid interval" />
          </el-form-item>
          <el-form-item v-if="activeData.layout==='rowFormItem'" label="layout mode">
            <el-radio-group v-model="activeData.type">
              <el-radio-button label="default" />
              <el-radio-button label="flex" />
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData.justify!==undefined&&activeData.type==='flex'" label="flex">
            <el-select v-model="activeData.justify" placeholder="Please select horizontal arrangement" :style="{width: '100%'}">
              <el-option
                v-for="(item, index) in justifyOptions"
                :key="index"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.align!==undefined&&activeData.type==='flex'" label="ailign">
            <el-radio-group v-model="activeData.align">
              <el-radio-button label="top" />
              <el-radio-button label="middle" />
              <el-radio-button label="bottom" />
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData.labelWidth!==undefined" label="label width">
            <el-input v-model.number="activeData.labelWidth" type="number" placeholder="Please enter label width" />
          </el-form-item>
          <el-form-item v-if="activeData.style&&activeData.style.width!==undefined" label="组件宽度">
            <el-input v-model="activeData.style.width" placeholder="Please enter component width" clearable />
          </el-form-item>
          <el-form-item v-if="activeData.vModel!==undefined" label="default value">
            <el-input
              :value="setDefaultValue(activeData.defaultValue)"
              placeholder="Please enter the default value"
              @input="onDefaultValueInput"
            />
          </el-form-item>
          <el-form-item v-if="activeData.tag==='el-checkbox-group'" label="at least should be selected">
            <el-input-number
              :value="activeData.min"
              :min="0"
              placeholder="At least should be selected"
              @input="$set(activeData, 'min', $event?$event:undefined)"
            />
          </el-form-item>
          <el-form-item v-if="activeData.tag==='el-checkbox-group'" label="up to optional">
            <el-input-number
              :value="activeData.max"
              :min="0"
              placeholder="Maximum optional"
              @input="$set(activeData, 'max', $event?$event:undefined)"
            />
          </el-form-item>
          <el-form-item v-if="activeData.prepend!==undefined" label="前缀">
            <el-input v-model="activeData.prepend" placeholder="Please enter a prefix" />
          </el-form-item>
          <el-form-item v-if="activeData.append!==undefined" label="后缀">
            <el-input v-model="activeData.append" placeholder="Please enter suffix" />
          </el-form-item>
          <el-form-item v-if="activeData['prefix-icon']!==undefined" label="前图标">
            <el-input v-model="activeData['prefix-icon']" placeholder="Please enter the name of the front icon">
              <el-button slot="append" icon="el-icon-thumb" @click="openIconsDialog('prefix-icon')">
                选择
              </el-button>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData['suffix-icon'] !== undefined" label="后图标">
            <el-input v-model="activeData['suffix-icon']" placeholder="Please enter the icon name">
              <el-button slot="append" icon="el-icon-thumb" @click="openIconsDialog('suffix-icon')">
                选择
              </el-button>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-cascader'" label="option separator">
            <el-input v-model="activeData.separator" placeholder="Please enter the option separator" />
          </el-form-item>
          <el-form-item v-if="activeData.autosize !== undefined" label="最小行数">
            <el-input-number v-model="activeData.autosize.minRows" :min="1" placeholder="最小行数" />
          </el-form-item>
          <el-form-item v-if="activeData.autosize !== undefined" label="最大行数">
            <el-input-number v-model="activeData.autosize.maxRows" :min="1" placeholder="最大行数" />
          </el-form-item>
          <el-form-item v-if="activeData.min !== undefined" label="minimum">
            <el-input-number v-model="activeData.min" placeholder="minimum" />
          </el-form-item>
          <el-form-item v-if="activeData.max !== undefined" label="Max">
            <el-input-number v-model="activeData.max" placeholder="Max" />
          </el-form-item>
          <el-form-item v-if="activeData.step !== undefined" label="步长">
            <el-input-number v-model="activeData.step" placeholder="步数" />
          </el-form-item>
          <el-form-item v-if="activeData.tag === 'el-input-number'" label="精度">
            <el-input-number v-model="activeData.precision" :min="0" placeholder="精度" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-input-number'" label="button location">
            <el-radio-group v-model="activeData['controls-position']">
              <el-radio-button label="">
                default
              </el-radio-button>
              <el-radio-button label="right">
                Right
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData.maxlength !== undefined" label="max input">
            <el-input v-model="activeData.maxlength" placeholder="Please enter character length">
              <template slot="append">
                Characters
              </template>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData['active-text'] !== undefined" label="open prompt">
            <el-input v-model="activeData['active-text']" placeholder="Please enter an opening prompt" />
          </el-form-item>
          <el-form-item v-if="activeData['inactive-text'] !== undefined" label="Close prompt">
            <el-input v-model="activeData['inactive-text']" placeholder="Please enter a prompt to close" />
          </el-form-item>
          <el-form-item v-if="activeData['active-value'] !== undefined" label="开启值">
            <el-input
              :value="setDefaultValue(activeData['active-value'])"
              placeholder="Please enter the opening value"
              @input="onSwitchValueInput($event, 'active-value')"
            />
          </el-form-item>
          <el-form-item v-if="activeData['inactive-value'] !== undefined" label="关闭值">
            <el-input
              :value="setDefaultValue(activeData['inactive-value'])"
              placeholder="Please enter close value"
              @input="onSwitchValueInput($event, 'inactive-value')"
            />
          </el-form-item>
          <el-form-item
            v-if="activeData.type !== undefined && 'el-date-picker' === activeData.tag"
            label="Time type"
          >
            <el-select
              v-model="activeData.type"
              placeholder="Please select the time type"
              :style="{ width: '100%' }"
              @change="dateTypeChange"
            >
              <el-option
                v-for="(item, index) in dateOptions"
                :key="index"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.name !== undefined" label="file field name">
            <el-input v-model="activeData.name" placeholder="Please enter the field name of the uploaded file" />
          </el-form-item>
          <el-form-item v-if="activeData.accept !== undefined" label="file type">
            <el-select
              v-model="activeData.accept"
              placeholder="Please select the file type"
              :style="{ width: '100%' }"
              clearable
            >
              <el-option label="picture" value="image/*" />
              <el-option label="video" value="video/*" />
              <el-option label="audio" value="audio/*" />
              <el-option label="excel" value=".xls,.xlsx" />
              <el-option label="word" value=".doc,.docx" />
              <el-option label="pdf" value=".pdf" />
              <el-option label="txt" value=".txt" />
            </el-select>
          </el-form-item>
          <el-form-item v-if="activeData.fileSize !== undefined" label="file size">
            <el-input v-model.number="activeData.fileSize" placeholder="Please enter the file size">
              <el-select slot="append" v-model="activeData.sizeUnit" :style="{ width: '66px' }">
                <el-option label="KB" value="KB" />
                <el-option label="MB" value="MB" />
                <el-option label="GB" value="GB" />
              </el-select>
            </el-input>
          </el-form-item>
          <el-form-item v-if="activeData.action !== undefined" label="upload address">
            <el-input v-model="activeData.action" placeholder="Please enter the upload address" clearable />
          </el-form-item>
          <el-form-item v-if="activeData['list-type'] !== undefined" label="list type">
            <el-radio-group v-model="activeData['list-type']" size="small">
              <el-radio-button label="text">
                text
              </el-radio-button>
              <el-radio-button label="picture">
                picture
              </el-radio-button>
              <el-radio-button label="picture-card">
                picture-card
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item
            v-if="activeData.buttonText !== undefined"
            v-show="'picture-card' !== activeData['list-type']"
            label="Button text"
          >
            <el-input v-model="activeData.buttonText" placeholder="Please enter button text" />
          </el-form-item>
          <el-form-item v-if="activeData['range-separator'] !== undefined" label="分隔符">
            <el-input v-model="activeData['range-separator']" placeholder="Please enter a separator" />
          </el-form-item>
          <el-form-item v-if="activeData['picker-options'] !== undefined" label="时间段">
            <el-input
              v-model="activeData['picker-options'].selectableRange"
              placeholder="Please enter a time period"
            />
          </el-form-item>
          <el-form-item v-if="activeData.format !== undefined" label="time format">
            <el-input
              :value="activeData.format"
              placeholder="Please enter the time format"
              @input="setTimeValue($event)"
            />
          </el-form-item>
          <template v-if="['el-checkbox-group', 'el-radio-group', 'el-select'].indexOf(activeData.tag) > -1">
            <el-divider>选项</el-divider>
            <draggable
              :list="activeData.options"
              :animation="340"
              group="selectItem"
              handle=".option-drag"
            >
              <div v-for="(item, index) in activeData.options" :key="index" class="select-item">
                <div class="select-line-icon option-drag">
                  <i class="el-icon-s-operation" />
                </div>
                <el-input v-model="item.label" placeholder="选项名" size="small" />
                <el-input
                  placeholder="option value"
                  size="small"
                  :value="item.value"
                  @input="setOptionValue(item, $event)"
                />
                <div class="close-btn select-line-icon" @click="activeData.options.splice(index, 1)">
                  <i class="el-icon-remove-outline" />
                </div>
              </div>
            </draggable>
            <div style="margin-left: 20px;">
              <el-button
                style="padding-bottom: 0"
                icon="el-icon-circle-plus-outline"
                type="text"
                @click="addSelectItem"
              >
                Add options
              </el-button>
            </div>
            <el-divider />
          </template>

          <template v-if="['el-cascader'].indexOf(activeData.tag) > -1">
            <el-divider>Options</el-divider>
            <el-form-item label="data type">
              <el-radio-group v-model="activeData.dataType" size="small">
                <el-radio-button label="dynamic">
                  dynamic data
                </el-radio-button>
                <el-radio-button label="static">
                  Static data
                </el-radio-button>
              </el-radio-group>
            </el-form-item>

            <template v-if="activeData.dataType === 'dynamic'">
              <el-form-item label="label key name">
                <el-input v-model="activeData.labelKey" placeholder="Please enter the label key name" />
              </el-form-item>
              <el-form-item label="value key name">
                <el-input v-model="activeData.valueKey" placeholder="Please enter the value key name" />
              </el-form-item>
              <el-form-item label="child key name">
                <el-input v-model="activeData.childrenKey" placeholder="Please enter the child key name" />
              </el-form-item>
            </template>

            <el-tree
              v-if="activeData.dataType === 'static'"
              draggable
              :data="activeData.options"
              node-key="id"
              :expand-on-click-node="false"
              :render-content="renderContent"
            />
            <div v-if="activeData.dataType === 'static'" style="margin-left: 20px">
              <el-button
                style="padding-bottom: 0"
                icon="el-icon-circle-plus-outline"
                type="text"
                @click="addTreeItem"
              >
                Add parent
              </el-button>
            </div>
            <el-divider />
          </template>

          <el-form-item v-if="activeData.optionType !== undefined" label="option style">
            <el-radio-group v-model="activeData.optionType">
              <el-radio-button label="default">
                default
              </el-radio-button>
              <el-radio-button label="button">
                Button
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData['active-color'] !== undefined" label="open color">
            <el-color-picker v-model="activeData['active-color']" />
          </el-form-item>
          <el-form-item v-if="activeData['inactive-color'] !== undefined" label="Close color">
            <el-color-picker v-model="activeData['inactive-color']" />
          </el-form-item>

          <el-form-item v-if="activeData['allow-half'] !== undefined" label="Allow half selection">
            <el-switch v-model="activeData['allow-half']" />
          </el-form-item>
          <el-form-item v-if="activeData['show-text'] !== undefined" label="辅助文字">
            <el-switch v-model="activeData['show-text']" @change="rateTextChange" />
          </el-form-item>
          <el-form-item v-if="activeData['show-score'] !== undefined" label="display score">
            <el-switch v-model="activeData['show-score']" @change="rateScoreChange" />
          </el-form-item>
          <el-form-item v-if="activeData['show-stops'] !== undefined" label="display break point">
            <el-switch v-model="activeData['show-stops']" />
          </el-form-item>
          <el-form-item v-if="activeData.range !== undefined" label="range selection">
            <el-switch v-model="activeData.range" @change="rangeChange" />
          </el-form-item>
          <el-form-item
            v-if="activeData.border !== undefined && activeData.optionType === 'default'"
            label="Whether with border"
          >
            <el-switch v-model="activeData.border" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-color-picker'" label="color format">
            <el-select
              v-model="activeData['color-format']"
              placeholder="Please select a color format"
              :style="{ width: '100%' }"
              @change="colorFormatChange"
            >
              <el-option
                v-for="(item, index) in colorFormatOptions"
                :key="index"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item
            v-if="activeData.size !== undefined &&
              (activeData.optionType === 'button' ||
              activeData.border ||
              activeData.tag === 'el-color-picker')"
            label="选项尺寸"
          >
            <el-radio-group v-model="activeData.size">
              <el-radio-button label="medium">
                中等
              </el-radio-button>
              <el-radio-button label="small">
                较小
              </el-radio-button>
              <el-radio-button label="mini">
                迷你
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="activeData['show-word-limit'] !== undefined" label="input statistics">
            <el-switch v-model="activeData['show-word-limit']" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-input-number'" label="strict step count">
            <el-switch v-model="activeData['step-strictly']" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-cascader'" label="whether multiple choices">
            <el-switch v-model="activeData.props.props.multiple" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-cascader'" label="Show full path">
            <el-switch v-model="activeData['show-all-levels']" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-cascader'" label="Can filter">
            <el-switch v-model="activeData.filterable" />
          </el-form-item>
          <el-form-item v-if="activeData.clearable !== undefined" label="能否清空">
            <el-switch v-model="activeData.clearable" />
          </el-form-item>
          <el-form-item v-if="activeData.showTip !== undefined" label="show tips">
            <el-switch v-model="activeData.showTip" />
          </el-form-item>
          <el-form-item v-if="activeData.multiple !== undefined" label="multiple choice files">
            <el-switch v-model="activeData.multiple" />
          </el-form-item>
          <el-form-item v-if="activeData['auto-upload'] !== undefined" label="automatic upload">
            <el-switch v-model="activeData['auto-upload']" />
          </el-form-item>
          <el-form-item v-if="activeData.readonly !== undefined" label="is read-only">
            <el-switch v-model="activeData.readonly" />
          </el-form-item>
          <el-form-item v-if="activeData.disabled !== undefined" label="is disabled">
            <el-switch v-model="activeData.disabled" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-select'" label="is searchable">
            <el-switch v-model="activeData.filterable" />
          </el-form-item>
          <el-form-item v-if="activeData.tag ==='el-select'" label="whether multiple choices">
            <el-switch v-model="activeData.multiple" @change="multipleChange" />
          </el-form-item>
          <el-form-item v-if="activeData.required !== undefined" label="required">
            <el-switch v-model="activeData.required" />
          </el-form-item>

          <template v-if="activeData.layoutTree">
            <el-divider>Layout structure tree</el-divider>
            <el-tree
              :data="[activeData]"
              :props="layoutTreeProps"
              node-key="renderKey"
              default-expand-all
              draggable
            >
              <span slot-scope="{ node, data }">
                <span class="node-label">
                  <svg-icon class="node-icon" :icon-class="data.tagIcon" />
                  {{ node.label }}
                </span>
              </span>
            </el-tree>
          </template>

          <template v-if="activeData.layout === 'colFormItem'">
            <el-divider>Regular verification</el-divider>
            <div
              v-for="(item, index) in activeData.regList"
              :key="index"
              class="reg-item"
            >
              <span class="close-btn" @click="activeData.regList.splice(index, 1)">
                <i class="el-icon-close" />
              </span>
              <el-form-item label="expression">
                <el-input v-model="item.pattern" placeholder="Please enter regular" />
              </el-form-item>
              <el-form-item label="error prompt" style="margin-bottom:0">
                <el-input v-model="item.message" placeholder="Please enter an error message" />
              </el-form-item>
            </div>
            <div style="margin-left: 20px">
              <el-button icon="el-icon-circle-plus-outline" type="text" @click="addReg">
                Add rule
              </el-button>
            </div>
          </template>
        </el-form>
        <!-- Form properties -->
        <el-form v-show="currentTab === 'form'" size="small" label-width="90px">
          <el-form-item label="form name">
            <el-input v-model="formConf.formRef" placeholder="Please enter the form name (ref)" />
          </el-form-item>
          <el-form-item label="Form model">
            <el-input v-model="formConf.formModel" placeholder="Please enter the data model" />
          </el-form-item>
          <el-form-item label="Verification model">
            <el-input v-model="formConf.formRules" placeholder="Please enter the verification model" />
          </el-form-item>
          <el-form-item label="form size">
            <el-radio-group v-model="formConf.size">
              <el-radio-button label="medium">
                medium
              </el-radio-button>
              <el-radio-button label="small">
                Smaller
              </el-radio-button>
              <el-radio-button label="mini">
                Mini
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="label alignment">
            <el-radio-group v-model="formConf.labelPosition">
              <el-radio-button label="left">
                Align left
              </el-radio-button>
              <el-radio-button label="right">
                Right
              </el-radio-button>
              <el-radio-button label="top">
                Align top
              </el-radio-button>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="label width">
            <el-input-number v-model="formConf.labelWidth" placeholder="label width" />
          </el-form-item>
          <el-form-item label="Grid spacing">
            <el-input-number v-model="formConf.gutter" :min="0" placeholder="Grid spacing" />
          </el-form-item>
          <el-form-item label="Disable Form">
            <el-switch v-model="formConf.disabled" />
          </el-form-item>
          <el-form-item label="form button">
            <el-switch v-model="formConf.formBtns" />
          </el-form-item>
          <el-form-item label="Show unselected component borders">
            <el-switch v-model="formConf.unFocusedComponentBorder" />
          </el-form-item>
        </el-form>
      </el-scrollbar>
    </div>

    <treeNode-dialog :visible.sync="dialogVisible" title="Add option" @commit="addNode" />
    <icons-dialog :visible.sync="iconsVisible" :current="activeData[currentIconModel]" @select="setIcon" />
  </div>
</template>

<script>
import { isArray } from 'util'
import TreeNodeDialog from './TreeNodeDialog'
import { isNumberStr } from '@/utils/index'
import IconsDialog from './IconsDialog'
import {
  inputComponents,
  selectComponents,
  // eslint-disable-next-line no-unused-vars
  layoutComponents
} from '@/utils/generator/config'

const dateTimeFormat = {
  date: 'yyyy-MM-dd',
  week: 'yyyy 第 WW 周',
  month: 'yyyy-MM',
  year: 'yyyy',
  datetime: 'yyyy-MM-dd HH:mm:ss',
  daterange: 'yyyy-MM-dd',
  monthrange: 'yyyy-MM',
  datetimerange: 'yyyy-MM-dd HH:mm:ss'
}

export default {
  components: {
    TreeNodeDialog,
    IconsDialog
  },
  // eslint-disable-next-line vue/require-prop-types
  props: ['showField', 'activeData', 'formConf'],
  data() {
    return {
      currentTab: 'field',
      currentNode: null,
      dialogVisible: false,
      iconsVisible: false,
      currentIconModel: null,
      dateTypeOptions: [
        {
          label: 'date',
          value: 'date'
        },
        {
          label: 'week',
          value: 'week'
        },
        {
          label: 'month',
          value: 'month'
        },
        {
          label: 'year',
          value: 'year'
        },
        {
          label: 'datetime',
          value: 'datetime'
        }
      ],
      dateRangeTypeOptions: [
        {
          label: 'daterange',
          value: 'daterange'
        },
        {
          label: 'monthrange',
          value: 'monthrange'
        },
        {
          label: 'datetimerange',
          value: 'datetimerange'
        }
      ],
      colorFormatOptions: [
        {
          label: 'hex',
          value: 'hex'
        },
        {
          label: 'rgb',
          value: 'rgb'
        },
        {
          label: 'rgba',
          value: 'rgba'
        },
        {
          label: 'hsv',
          value: 'hsv'
        },
        {
          label: 'hsl',
          value: 'hsl'
        }
      ],
      justifyOptions: [
        {
          label: 'start',
          value: 'start'
        },
        {
          label: 'end',
          value: 'end'
        },
        {
          label: 'center',
          value: 'center'
        },
        {
          label: 'space-around',
          value: 'space-around'
        },
        {
          label: 'space-between',
          value: 'space-between'
        }
      ],
      layoutTreeProps: {
        label(data, node) {
          return data.componentName || `${data.label}: ${data.vModel}`
        }
      }
    }
  },
  computed: {
    documentLink() {
      return (
        this.activeData.document ||
        'https://element.eleme.cn/#/zh-CN/component/installation'
      )
    },
    dateOptions() {
      if (
        this.activeData.type !== undefined &&
        this.activeData.tag === 'el-date-picker'
      ) {
        if (this.activeData['start-placeholder'] === undefined) {
          return this.dateTypeOptions
        }
        return this.dateRangeTypeOptions
      }
      return []
    },
    tagList() {
      return [
        {
          label: 'Input component',
          options: inputComponents
        },
        {
          label: 'Selected component',
          options: selectComponents
        }
      ]
    }
  },
  methods: {
    addReg() {
      this.activeData.regList.push({
        pattern: '',
        message: ''
      })
    },
    addSelectItem() {
      this.activeData.options.push({
        label: '',
        value: ''
      })
    },
    addTreeItem() {
      ++this.idGlobal
      this.dialogVisible = true
      this.currentNode = this.activeData.options
    },
    renderContent(h, { node, data, store }) {
      return (
        <div class='custom-tree-node'>
          <span>{node.label}</span>
          <span class='node-operation'>
            <i on-click={() => this.append(data)}
              class='el-icon-plus'
              title='Add'
            ></i>
            <i on-click={() => this.remove(node, data)}
              class='el-icon-delete'
              title='Delete'
            ></i>
          </span>
        </div>
      )
    },
    append(data) {
      if (!data.children) {
        this.$set(data, 'children', [])
      }
      this.dialogVisible = true
      this.currentNode = data.children
    },
    remove(node, data) {
      const { parent } = node
      const children = parent.data.children || parent.data
      const index = children.findIndex(d => d.id === data.id)
      children.splice(index, 1)
    },
    addNode(data) {
      this.currentNode.push(data)
    },
    setOptionValue(item, val) {
      item.value = isNumberStr(val) ? +val : val
    },
    setDefaultValue(val) {
      if (Array.isArray(val)) {
        return val.join(',')
      }
      if (['string', 'number'].indexOf(val) > -1) {
        return val
      }
      if (typeof val === 'boolean') {
        return `${val}`
      }
      return val
    },
    onDefaultValueInput(str) {
      if (isArray(this.activeData.defaultValue)) {
        // 数组
        this.$set(
          this.activeData,
          'defaultValue',
          str.split(',').map(val => (isNumberStr(val) ? +val : val))
        )
      } else if (['true', 'false'].indexOf(str) > -1) {
        // Boolean
        this.$set(this.activeData, 'defaultValue', JSON.parse(str))
      } else {
        // Strings and numbers
        this.$set(
          this.activeData,
          'defaultValue',
          isNumberStr(str) ? +str : str
        )
      }
    },
    onSwitchValueInput(val, name) {
      if (['true', 'false'].indexOf(val) > -1) {
        this.$set(this.activeData, name, JSON.parse(val))
      } else {
        this.$set(this.activeData, name, isNumberStr(val) ? +val : val)
      }
    },
    setTimeValue(val, type) {
      const valueFormat = type === 'week' ? dateTimeFormat.date : val
      this.$set(this.activeData, 'defaultValue', null)
      this.$set(this.activeData, 'value-format', valueFormat)
      this.$set(this.activeData, 'format', val)
    },
    spanChange(val) {
      this.formConf.span = val
    },
    multipleChange(val) {
      this.$set(this.activeData, 'defaultValue', val ? [] : '')
    },
    dateTypeChange(val) {
      this.setTimeValue(dateTimeFormat[val], val)
    },
    rangeChange(val) {
      this.$set(
        this.activeData,
        'defaultValue',
        val ? [this.activeData.min, this.activeData.max] : this.activeData.min
      )
    },
    rateTextChange(val) {
      if (val) this.activeData['show-score'] = false
    },
    rateScoreChange(val) {
      if (val) this.activeData['show-text'] = false
    },
    colorFormatChange(val) {
      this.activeData.defaultValue = null
      this.activeData['show-alpha'] = val.indexOf('a') > -1
      this.activeData.renderKey = +new Date() // update renderKey, re-render the component
    },
    openIconsDialog(model) {
      this.iconsVisible = true
      this.currentIconModel = model
    },
    setIcon(val) {
      this.activeData[this.currentIconModel] = val
    },
    tagChange(tagIcon) {
      let target = inputComponents.find(item => item.tagIcon === tagIcon)
      if (!target) target = selectComponents.find(item => item.tagIcon === tagIcon)
      this.$emit('tag-change', target)
    }
  }
}
</script>

<style lang="scss" scoped>
.right-board {
  width: 350px;
  position: absolute;
  right: 0;
  top: 0;
  padding-top: 3px;
  .field-box {
    position: relative;
    height: calc(100vh - 42px);
    box-sizing: border-box;
    overflow: hidden;
  }
  .el-scrollbar {
    height: 100%;
  }
}
.select-item {
  display: flex;
  border: 1px dashed #fff;
  box-sizing: border-box;
  & .close-btn {
    cursor: pointer;
    color: #f56c6c;
  }
  & .el-input + .el-input {
    margin-left: 4px;
  }
}
.select-item + .select-item {
  margin-top: 4px;
}
.select-item.sortable-chosen {
  border: 1px dashed #409eff;
}
.select-line-icon {
  line-height: 32px;
  font-size: 22px;
  padding: 0 4px;
  color: #777;
}
.option-drag {
  cursor: move;
}
.time-range {
  .el-date-editor {
    width: 227px;
  }
  ::v-deep .el-icon-time {
    display: none;
  }
}
.document-link {
  position: absolute;
  display: block;
  width: 26px;
  height: 26px;
  top: 0;
  left: 0;
  cursor: pointer;
  background: #409eff;
  z-index: 1;
  border-radius: 0 0 6px 0;
  text-align: center;
  line-height: 26px;
  color: #fff;
  font-size: 18px;
}
.node-label{
  font-size: 14px;
}
.node-icon{
  color: #bebfc3;
}
</style>
