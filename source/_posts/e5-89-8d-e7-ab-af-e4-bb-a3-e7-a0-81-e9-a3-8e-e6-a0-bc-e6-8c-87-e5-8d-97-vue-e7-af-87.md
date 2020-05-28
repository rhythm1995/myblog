---
title: 前端代码风格指南——vue篇
url: 214.html
id: 214
categories:
  - 前端
date: 2018-11-09 01:29:45
tags:
---

是我常用的一套开发标准，借鉴、删减、自创的一些规则，保留的基本都是比较实用的。用于统一代码风格。并给出了每条标准的理由。

* * *

*   组件名为多个单词（必须） \> 理由：这样做可以避免跟现有的以及未来的 HTML 元素相冲突，因为所有的 HTML 元素名称都是单个单词的。

    /* 不推荐的 */
    Vue.component('todo', {
      // ...
    })
    export default {
      name: 'Todo',
      // ...
    }
    
    /* 推荐的 */
    Vue.component('todo-item', {
      // ...
    })
    export default {
      name: 'TodoItem',
      // ...
    }
    

*   prop 的定义应该尽量详细，至少需要指定其类型（必须） > 理由：一目了然的知道prop是什么，并且在类型报错时可以顺利定位。

    /* 不推荐的，可以在demo和原型中使用 */
    props: ['status']
    
    /* 推荐的 */
    props: {
      status: String
    }
    
    /* 最好的 */
    props: {
      status: {
        type: String,
        required: true,
        validator(value) {
          return [
            'syncing',
            'synced',
            'version-conflict',
            'error'
          ].indexOf(value) !== -1
        }
      }
    }
    

*   为v-for设置index与key > 理由：方便维护内部组件及其子树的状态

    /* 不推荐的 */
    <ul>
      <li v-for="todo in todos">
        { { todo.text }}
      </li>
    </ul>
    
    /* 推荐的 */
    <ul>
      <li
        v-for="todo in todos"
        :key="todo.id"
      >
        { { todo.text }}
      </li>
    </ul>
    

*   组件命名（可选）
    
    *   单文件组件的文件名应该要么始终是单词大写开头 (PascalCase)，要么始终是横线连接 (kebab-case) > 理由：单词大写开头对于代码编辑器的自动补全最为友好，而对于模板横线连接可以避免大小写不敏感的问题。
    
        /* 不推荐的 */
        <ul>
        <li v-for="todo in todos">
          { { todo.text }}
        </li>
        </ul>
        
        /* 推荐的 */
        <ul>
        <li
          v-for="todo in todos"
          :key="todo.id"
        >
          { { todo.text }}
        </li>
        </ul>
        
    
    *   只应该拥有单个活跃实例的组件应该以 The 前缀命名，以示其唯一性 > 理由：增加项目可读性。
    
        /* 不推荐的 */
        components/
        |- Heading.vue
        |- MySidebar.vue
        
        /* 推荐的 */
        components/
        |- TheHeading.vue
        |- TheSidebar.vue
        
    
    *   和父组件紧密耦合的子组件应该以父组件名作为前缀命名 \> 理由：增加项目可读性与层级性。
    
        /* 不推荐的 */
        components/
        |- TodoList.vue
        |- TodoItem.vue
        |- TodoButton.vue
        
        /* 推荐的 */
        components/
        |- TodoList.vue
        |- TodoListItem.vue
        |- TodoListItemButton.vue
        
    
    *   组件名应该以高级别的 (通常是一般化描述的) 单词开头，以描述性的修饰词结尾 \> 理由：可以首先知道这个组件大概是什么东西，然后知道细节，此外与自然语法相反的这种描述是为了减少连接词的数量，否则组件名会变得很长
    
        /* 不推荐的 */
        components/
        |- ClearSearchButton.vue
        |- ExcludeFromSearchInput.vue
        |- LaunchOnStartupCheckbox.vue
        |- RunSearchButton.vue
        |- SearchInput.vue
        |- TermsCheckbox.vue
        
        /* 推荐的 */
        components/
        |- SearchButtonClear.vue
        |- SearchButtonRun.vue
        |- SearchInputQuery.vue
        |- SearchInputExcludeGlob.vue
        |- SettingsCheckboxTerms.vue
        |- SettingsCheckboxLaunchOnStartup.vue
        
    
    *   对于绝大多数项目来说，在单文件组件和字符串模板中组件名应该总是 PascalCase 的——但是在 DOM 模板中总是 kebab-case 的 > 理由：编辑器可以自动补全组件名，大驼峰保证了视觉的易识别性，但HTML是大小写不敏感，所以要使用横线连接式。
    
        /* 不推荐的 */
        <!-- 在单文件组件和字符串模板中 -->
        <mycomponent/>
        <!-- 在单文件组件和字符串模板中 -->
        <myComponent/>
        <!-- 在 DOM 模板中 -->
        <MyComponent></MyComponent>
        
        /* 推荐的 */
        <!-- 在单文件组件和字符串模板中 -->
        <MyComponent/>
        <!-- 在 DOM 模板中 -->
        <my-component></my-component>
        
    
*   在声明 prop 的时候，其命名应该始终使用 camelCase，而在模板和 JSX 中应该始终使用 kebab-case（建议）
    
    > 理由：遵循js和html的命名习惯。
    

    /* 不推荐的 */
    props: {
      'greeting-text': String
    }
    <WelcomeMessage greetingText="hi"/>
    
    /* 推荐的 */
    props: {
      greetingText: String
    }
    <WelcomeMessage greeting-text="hi"/>
    

*   模板中的属性多于3个时候，分行写属性（推荐） > 理由：首先太多的话分行不好读，其次这样比较好看；少的话即使在一行读起来问题不大，而且一行可以少点行数

    /* 不推荐的 */
    <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
    </el-option>
    
    /* 推荐的 */
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
    
      <el-button plain @click="open">可自动关闭</el-button>
    

*   指令全部使用缩写 ，用 : 表示 v-bind: 和用 @ 表示 v-on:（推荐） > 理由：一样的东西能少些为什么要多写。

    /* 不推荐的 */
      <input
        v-bind:value="newTodoText"
        @focus="onFocus"
      >
    
    /* 推荐的 */
      <input
        :value="newTodoText"
        @focus="onFocus"
      >
    
    /* 最差的 */
      <input
        v-on:input="onInput"
        @focus="onFocus"
      >