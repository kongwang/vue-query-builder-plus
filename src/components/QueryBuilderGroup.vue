<template>
  <div class="rules-group-container"> 
      <div class="rules-group-header">
          <div class="btn-group pull-right group-actions"> 
              <el-button-group>
                  <el-button size="mini" type="success" icon="el-icon-plus" @click="addRule">Add Rule</el-button>
                  <el-button size="mini" type="success" icon="el-icon-circle-plus-outline" v-if="this.depth < this.maxDepth" @click="addGroup">Add Group</el-button>
                  <el-button v-if="this.depth > 1" @click="remove" size="mini" type="danger" icon="el-icon-close">Delete</el-button>
              </el-button-group>
          </div>
          <div class="btn-group group-conditions">
            <el-radio-group v-model="query.logicalOperator" size="mini">
              <el-radio-button label="AND" :disabled="!hasMultipleRule">AND</el-radio-button>
              <el-radio-button label="OR" :disabled="!hasMultipleRule">OR</el-radio-button>
            </el-radio-group>
          </div>
          
      </div>
      <div class="rules-group-body"> 
        <div class="rules-list">
          <component
            v-for="(child, index) in query.children"
            :key="index"
            :is="child.type"
            :type="child.type"
            :query.sync="child.query"
            :ruleTypes="ruleTypes"
            :rules="rules"
            :index="index"
            :maxDepth="maxDepth"
            :depth="depth + 1"
            v-on:child-deletion-requested="removeChild">
          </component>
        </div>
      </div>
  </div>
</template>

<script>
import QueryBuilderRule from "./QueryBuilderRule.vue";
import deepClone from "../utils/deepClone.js";

export default {
  name: "query-builder-group",

  components: {
    QueryBuilderRule
  },

  props: ["ruleTypes", "type", "query", "rules", "index", "maxDepth", "depth"],

  methods: {
    addRule() {
      const updated_query = deepClone(this.query);
      const child = {
        type: "query-builder-rule",
        query: {
          rule: this.rules[0].id,
          selectedOperator: this.rules[0].operators[0].value,
          value: null
        }
      };
      updated_query.children.push(child);
      this.$emit("update:query", updated_query);
    },

    addGroup() {
      const updated_query = deepClone(this.query);
      if (this.depth < this.maxDepth) {
        updated_query.children.push({
          type: "query-builder-group",
          query: {
            logicalOperator: "AND",
            children: [
              {
                type: "query-builder-rule",
                query: {
                  rule: this.rules[0].id,
                  selectedOperator: this.rules[0].operators[0].value,
                  value: null
                }
              }
            ]
          }
        });
        this.$emit("update:query", updated_query);
      }
    },

    remove() {
      this.$emit("child-deletion-requested", this.index);
    },

    removeChild(index) {
      const updated_query = deepClone(this.query);
      updated_query.children.splice(index, 1);
      this.$emit("update:query", updated_query);
    }
  },

  data() {
    return {};
  },
  computed: {
    hasMultipleRule() {
      return this.query.children.length > 1;
    }
  }
};
</script>
