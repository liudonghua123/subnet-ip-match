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
              <a-form-item>
                <a-textarea
                  v-decorator="['result']"
                  placeholder="Result"
                  :rows="5"
                />
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
import { subnet, cidrSubnet } from "browserify-ip";

export default {
  beforeCreate() {
    this.form = this.$form.createForm(this);
  },
  data: () => {
    return {};
  },
  methods: {
    onSearch(ip) {
      console.info(`search ip ${ip}`);
      this.form.validateFields((err, values) => {
        if (!err) {
          console.log("Received values of form: ", values);
          const { searchIp, searchList } = values;
          // https://regex101.com/r/3ZI0Tq/1/
          const cidrSubnetRegexp = /(?<ip>[\d.:a-f]{3,})(?:[\s/,;]+)(?<cidr>\d{1,2})/gim;
          const maskSubnetRegexp = /(?<ip>[\d.:a-f]{3,})(?:[\s/,;]+)(?<mask>[\d.:a-f]{3,})/gim;
          const cidrSubnets = [];
          const maskSubnets = [];
          let matches = searchList.matchAll(cidrSubnetRegexp);
          for (const match of matches) {
            const {
              groups: { ip, cidr }
            } = match;
            cidrSubnets.push({ ip, cidr });
          }
          matches = searchList.matchAll(maskSubnetRegexp);
          for (const match of matches) {
            const {
              groups: { ip, mask }
            } = match;
            maskSubnets.push({ ip, mask });
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
          const result = [
            ...matchedCidrSubnets.map(item => `${item.ip}/${item.cidr}`),
            ...matchedMaskSubnets.map(item => `${item.ip},${item.mask}`)
          ].join("\n");
          this.form.setFieldsValue({ result });
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