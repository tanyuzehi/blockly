<script setup>
import { onMounted, ref } from "vue";

// Blockly 核心：主要的 Blockly 库，它定义了基本的 Blockly UI 和逻辑。
import * as Blockly from "blockly/core";
// 内置块定义：循环、逻辑、数学和字符串操作等常见块。
import * as libraryBlocks from "blockly/blocks";
// JavaScript 生成器：将块转换为 JavaScript，并包含所有内置块的块生成器。
import { javascriptGenerator } from "blockly/javascript";
import { pythonGenerator } from "blockly/python";
import { phpGenerator } from "blockly/php";
// 英语语言文件：内置块和 Blockly UI 上所有消息的字符串表（英语）。
import * as Zh from "blockly/msg/zh-hans";

Blockly.setLocale(Zh);

let workspace = null;
const toolbox = `<xml>
          <category name="逻辑" colour="%{BKY_LOGIC_HUE}">
            <block type="controls_if"></block>
            <block type="logic_compare"></block>
            <block type="logic_operation"></block>
            <block type="logic_negate"></block>
            <block type="logic_boolean"></block>
          </category>
          <category name="循环" colour="%{BKY_LOOPS_HUE}">
            <block type="controls_repeat_ext">
              <value name="TIMES">
                <block type="math_number">
                  <field name="NUM">10</field>
                </block>
              </value>
            </block>
            <block type="controls_whileUntil"></block>
          </category>
          <category name="数学" colour="%{BKY_MATH_HUE}">
            <block type="math_number">
              <field name="NUM">123</field>
            </block>
            <block type="math_arithmetic"></block>
            <block type="math_single"></block>
          </category>
          <category name="文本" colour="%{BKY_TEXTS_HUE}">
            <block type="text"></block>
            <block type="text_length"></block>
            <block type="text_print"></block>
          </category>
          <category name="变量" custom="VARIABLE" colour="%{BKY_VARIABLES_HUE}">
            </category>
        </xml>`;
/*{
  kind: "categoryToolbox",
  contents: [
    {
      kind: "category",
      name: "Logic",
      categorystyle: "logic_category",
      contents: [
        {
          kind: "block",
          type: "controls_if",
        },
        {
          kind: "block",
          type: "logic_compare",
        },
        {
          kind: "block",
          type: "logic_operation",
        },
        {
          kind: "block",
          type: "logic_negate",
        },
        {
          kind: "block",
          type: "logic_boolean",
        },
      ],
    },
    {
      kind: "category",
      name: "Loops",
      categorystyle: "loop_category",
      contents: [
        {
          kind: "block",
          type: "controls_repeat_ext",
          inputs: {
            TIMES: {
              block: {
                type: "math_number",
                fields: {
                  NUM: 10,
                },
              },
            },
          },
        },
        {
          kind: "block",
          type: "controls_whileUntil",
        },
      ],
    },
    {
      kind: "category",
      name: "Math",
      categorystyle: "math_category",
      contents: [
        {
          kind: "block",
          type: "math_number",
          fields: {
            NUM: 123,
          },
        },
        {
          kind: "block",
          type: "math_arithmetic",
        },
        {
          kind: "block",
          type: "math_single",
        },
      ],
    },
    {
      kind: "category",
      name: "Text",
      categorystyle: "text_category",
      contents: [
        {
          kind: "block",
          type: "text",
        },
        {
          kind: "block",
          type: "text_length",
        },
        {
          kind: "block",
          type: "text_print",
        },
      ],
    },
    {
      kind: "category",
      name: "变量",
      categorystyle: "variable_category",
      custom: "VARIABLE", // ✅ 告诉 Blockly 使用内建变量逻辑
    },
  ],
};*/

/**
 * 初始化Blockly
 */
const initBlockly = () => {
  workspace = Blockly.inject("blocklyDiv", {
    toolbox: toolbox,
  });
  workspace.addChangeListener(updateCode);
};
/**
 * 实时生成代码
 */
const updateCode = () => {
  console.log(javascriptGenerator.workspaceToCode(workspace));
};
const resizeDom = () => {
  const blocklyArea = document.getElementById("blocklyArea");
  const blocklyDiv = document.getElementById("blocklyDiv");
  const onresize = function (e) {
    let element = blocklyArea;
    let x = 0;
    let y = 0;
    do {
      x += element.offsetLeft;
      y += element.offsetTop;
      element = element.offsetParent;
    } while (element);
    // Position blocklyDiv over blocklyArea.
    blocklyDiv.style.left = x + "px";
    blocklyDiv.style.top = y + "px";
    blocklyDiv.style.width = blocklyArea.offsetWidth + "px";
    blocklyDiv.style.height = blocklyArea.offsetHeight + "px";
    Blockly.svgResize(workspace);
  };
  window.addEventListener("resize", onresize, false);
  onresize();
};

/**
 * 生成代码
 */
const handleGenerateCode = (type) => {
  switch (type) {
    case "JavaScript":
      console.log(javascriptGenerator.workspaceToCode(workspace));
      break;
    case "Python":
      console.log(pythonGenerator.workspaceToCode(workspace));
      break;
    case "PHP":
      console.log(phpGenerator.workspaceToCode(workspace));
      break;
  }
};

/**
 * 保存代码
 */
const handleSaveCode = () => {
  // 序列化状态
  const state = JSON.stringify(
    Blockly.serialization.workspaces.save(workspace),
  );
  // 然后将状态保存，例如保存到本地存储
  localStorage.setItem("workspace-state", state);
};

const handleLoadCode = () => {
  // 从某处获取保存的状态，例如从本地存储
  const state = JSON.parse(localStorage.getItem("workspace-state"));
  // 反序列化状态
  Blockly.serialization.workspaces.load(state, workspace);
};

/**
 * 获取变量
 */
const getVars = () => {
  const allVars = workspace.getVariableMap().getAllVariables();
  allVars.forEach((v) => {
    console.log("变量名:", v.name);
  });
};

/**
 * 运行代码
 */
const handleRunCode = () => {
  const code = javascriptGenerator.workspaceToCode(workspace);
  eval(code);
};

onMounted(() => {
  initBlockly();
  resizeDom();
});
</script>

<template>
  <div class="blockly">
    <div class="blockly-area" id="blocklyArea">
      <div id="blocklyDiv" style="height: 480px; width: 600px"></div>
    </div>
    <div class="button-group">
      <button @click="handleGenerateCode('JavaScript')">JavaScript代码</button>
      <button @click="handleGenerateCode('Python')">Python代码</button>
      <button @click="handleGenerateCode('PHP')">PHP代码</button>
      <button @click="handleSaveCode">保存代码</button>
      <button @click="handleLoadCode">加载代码</button>
      <button @click="getVars">获取变量</button>
      <button @click="handleRunCode">执行代码</button>
    </div>
  </div>
</template>

<style scoped>
.blockly {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;

  .blockly-area {
    width: 100%;
    height: 0;
    flex: 1;
  }
}
</style>
