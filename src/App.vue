<template>
  <a-layout id="components-layout-demo-top" class="layout">
    <a-layout-header>
      <div class="logo" />
      <a-menu
        theme="dark"
        mode="horizontal"
        :defaultSelectedKeys="['1']"
        :style="{ lineHeight: '64px' }"
      >
        <a-menu-item key="1">Subnet match</a-menu-item>
      </a-menu>
    </a-layout-header>
    <a-layout-content style="padding: 0 50px">
      <a-breadcrumb style="margin: 16px 0">
        <a-breadcrumb-item>Home</a-breadcrumb-item>
        <a-breadcrumb-item>Subnet match</a-breadcrumb-item>
      </a-breadcrumb>
      <div :style="{ background: '#fff', padding: '24px', minHeight: '280px' }">
        <a-form :form="form">
          <a-row>
            <a-col :span="24">
              <a-form-item>
                <a-input-search
                  v-decorator="[
                    'searchIp',
                    {
                      rules: [
                        {
                          required: true,
                          message: 'Please input your ip for searching!'
                        }
                      ]
                    }
                  ]"
                  placeholder="input ip address for searching"
                  @search="onSearch"
                  enterButton
                  size="large"
                />
              </a-form-item>
            </a-col>
          </a-row>
          <a-row>
            <a-col :span="24">
              <a-form-item>
                <a-input-search
                  v-decorator="['subnetsUrl']"
                  placeholder="input the url of the subnets file"
                  @search="onLoadSubnets"
                  enterButton="Load subnets resources"
                  size="large"
                />
              </a-form-item>
            </a-col>
          </a-row>
          <a-row>
            <a-col :span="24">
              <a-form-item>
                <a-textarea
                  v-decorator="[
                    'searchList',
                    {
                      rules: [
                        {
                          required: true,
                          message:
                            'Please input your list of subnet or cidrSubnet!'
                        }
                      ]
                    }
                  ]"
                  placeholder="input list of subnet or cidrSubnet like 1.1.0.0/16 or 1.1.0.0,255.255.0.0"
                  :rows="10"
                />
              </a-form-item>
            </a-col>
          </a-row>
          <a-row>
            <a-col :span="24">
              <a-form-item label="Results">
                <a-popover
                  v-for="(item, index) in results"
                  :key="index"
                  title="More info"
                >
                  <template slot="content">
                    <p>{{ JSON.stringify(item, null, 2) }}</p>
                  </template>
                  <a-tag color="green"
                    >{{ item.ip }} {{ item.cidr || item.mask }} #
                    {{ item.comments }}</a-tag
                  >
                </a-popover>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </a-layout-content>
    <a-layout-footer style="text-align: center">
      liudonghua123 ©2019 Created by Ant UED
    </a-layout-footer>
  </a-layout>
</template>
<script>
/*eslint no-console: "warn"*/
/*eslint no-unused-vars: "warn"*/
/*eslint no-useless-escape: "warn"*/
import { subnet, cidrSubnet } from "browserify-ip";

export default {
  beforeCreate() {
    this.form = this.$form.createForm(this);
  },
  data: () => {
    return { results: [] };
  },
  methods: {
    async onLoadSubnets(url) {
      try {
        const response = await fetch(url);
        // if HTTP-status is 200-299
        if (response.ok) {
          let searchList = await response.text();
          this.form.setFieldsValue({ searchList });
        } else {
          this.$message.error(
            `load resource failed with HTTP-Error: ${response.status}`
          );
        }
      } catch (error) {
        console.info(error);
        this.$message.error(
          `load resource failed with ${JSON.stringify(error)}`
        );
      }
    },
    onSearch(ip) {
      console.info(`search ip ${ip}`);
      this.form.validateFields((err, values) => {
        if (!err) {
          console.log("Received values of form: ", values);
          const { searchIp, searchList } = values;
          // https://regex101.com/r/3ZI0Tq/1/
          const cidrSubnetRegexp = /^[^\S\r\n]*(?<ip>[\d.:a-f]{3,})(?:[\t\f \/,;]+)(?<cidr>\d{1,2})(?:[\t\f \/,;#]+)(?<comments>.*)$/gim;
          const maskSubnetRegexp = /^[^\S\r\n]*(?<ip>[\d.:a-f]{3,})(?:[\t\f \/,;]+)(?<mask>[\d.:a-f]{3,})(?:[\t\f \/,;#]+)(?<comments>.*)$/gim;
          const cidrSubnets = [];
          const maskSubnets = [];
          let matches = searchList.matchAll(cidrSubnetRegexp);
          for (const match of matches) {
            const {
              groups: { ip, cidr, comments }
            } = match;
            cidrSubnets.push({ ip, cidr, comments });
          }
          matches = searchList.matchAll(maskSubnetRegexp);
          for (const match of matches) {
            const {
              groups: { ip, mask, comments }
            } = match;
            maskSubnets.push({ ip, mask, comments });
          }
          console.info(
            `parsed subnets\ncidrSubnets: ${JSON.stringify(
              cidrSubnets,
              null,
              2
            )}\nmaskSubnets: ${JSON.stringify(maskSubnets, null, 2)}`
          );
          const matchedCidrSubnets = cidrSubnets.filter(item =>
            cidrSubnet(`${item.ip}/${item.cidr}`).contains(searchIp)
          );
          const matchedMaskSubnets = maskSubnets.filter(item =>
            subnet(item.ip, item.mask).contains(searchIp)
          );

          const results =
            [
              ...matchedCidrSubnets.map(item => ({
                ...item,
                subnet: cidrSubnet(`${item.ip}/${item.cidr}`)
              })),
              ...matchedMaskSubnets.map(item => ({
                ...item,
                subnet: subnet(item.ip, item.mask)
              }))
            ] || [];
          this.results = results;
        }
      });
    }
  }
};
</script>
<style>
#components-layout-demo-top .logo {
  width: 120px;
  height: 30px;
  background: rgba(255, 255, 255, 0.2);
  margin: 16px 24px 16px 0;
  float: left;
}
</style>