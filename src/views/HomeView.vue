<script setup lang="ts">
import * as moduleBindings from '../spacetime_module_bindings/index';
import {ref, computed, onMounted} from "vue";
import type { TabsItem } from '@nuxt/ui'

const connection = ref<moduleBindings.DbConnection | null>();
const toast = useToast()
const rows = ref()

const serverUri = ref("");
const moduleName = ref("");

// TODO: Migrate to VueUse for cookies
function getCookie(cname: string) {
  let name = cname + "=";
  let decodedCookie = decodeURIComponent(document.cookie);
  let ca = decodedCookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) == ' ') {
      c = c.substring(1);
    }
    if (c.indexOf(name) == 0) {
      return c.substring(name.length, c.length);
    }
  }
  return null;
}

onMounted(() => {
  serverUri.value = getCookie("serverUri") ?? "ws://127.0.0.1:3000";
  moduleName.value = getCookie("moduleName") ?? "";
})

const
  handleConnectError = () => {
    toast.add({
      title: 'Error connecting to server',
      description: "Please ensure your Spacetime Server is running.",
      icon: "i-lucide-triangle-alert",
      color: "error"
    })
  }

const toSnakeCase = (str: string): string => {
  return str
    .replace(/([a-z0-9])([A-Z])/g, "$1_$2") // add underscore before capital letters
    .replace(/([A-Z])([A-Z][a-z])/g, "$1_$2") // handle consecutive capitals
    .toLowerCase();
};

const findTables = (): string[] => {
  return Object.keys(moduleBindings)
    .filter(key => key.endsWith("TableHandle"))
    .map(key => key.replace(/TableHandle$/, ""));
};

const tables = ref<any[]>([])

findTables().forEach((table) => {
  tables.value.push({
    name: table
  })
});

const tableSubscription = ref<any | null>();

const safeStringify = (obj: any, space = 2) =>
  JSON.stringify(obj, (_, value) =>
    typeof value === "bigint" ? value.toString() : value,
    space
  );

const selectTable = async(tableIndex: any) => {
  const tableName = tables.value[tableIndex].name;
  if (!connection.value) return ;

  selectedTable.value = tableName;

  rows.value = null;

  if (tableSubscription.value) {
    tableSubscription.value = null;
  }

  tableSubscription.value = connection.value
    .subscriptionBuilder()
    .onApplied((ctx: any) => {
      const tableKey = toSnakeCase(tableName);
      const table = ctx.db[tableKey];

      if (!table) {
        console.warn(`Table '${
tableKey
}' not found in ctx.db`);
        return ;
      }

      rows.value = table.iter();
    })
    .onError((errorCtx) => {
      toast.add({
        title: 'This table is not public',
        icon: "i-lucide-lock",
        color: "error"
      })
    })
    .subscribe(`SELECT * FROM ${
toSnakeCase(tableName)
}`);
}

const handleConnected = () => {
  if (!connection.value) return ;

  toast.add({
    title: 'Connected!',
    icon: "i-lucide-link",
    color: "success"
  })
}

const handleDisconnected = () => {
  connection.value = null;
  toast.add({
    title: 'Disconnected!',
    icon: "i-lucide-unlink",
    color: "success"
  })
}

const connect = () => {
  if (!serverUri.value || !moduleName.value) return ;

  // save values to cookie for next login
  document.cookie = `serverUri=${serverUri.value}`;
  document.cookie = `moduleName=${moduleName.value}`;

  // build connection
  connection.value = moduleBindings.DbConnection
    .builder()
    .withUri(serverUri.value)
    .withModuleName(moduleName.value)
    .onConnect(handleConnected)
    .onConnectError(handleConnectError)
    .onDisconnect(handleDisconnected)
    .build();
}

const disconnect = () => {
  if (connection.value) connection.value.disconnect();
}

if (connection.value) {
  const builder = new moduleBindings.SubscriptionBuilder(connection.value);

  builder.subscribeToAllTables();
}

const selectedTable = ref();
const selectedTableTab = ref();

const formattedRows = computed(() =>
  rows.value?.map((row) => {
    const formatted: Record<string, any> = {}
    for (const [key, value] of Object.entries(row)) {
      if (value === null || value === undefined) formatted[key] = ''
      else if (typeof value === 'object') formatted[key] = safeStringify(value)
      else formatted[key] = value
    }
    return formatted
  }) ?? []
)
</script>

<template>
  <div>
    <!--Header-->
    <div class="flex justify-between p-4 border-b border-accented">
      <div class="font-bold flex items-center">Spacetime Panel <UBadge label="v0.1.0" size="xs" variant="soft" class="ml-2"/></div>
      <div v-if="connection">
        <UBadge :label="'Connected to ' + serverUri" color="success" variant="soft" class="mr-2 rounded-full"/>
        <UButton label="Disconnect" color="neutral" @click="disconnect"/>
      </div>
      <div v-else>
        <UBadge label="Disconnected" color="error" variant="soft" class="rounded-full"/>
      </div>

    </div>

    <div class="flex">
      <!--  Sidebar-->
      <div class="border-r border-accented h-[calc(100vh-65px)] p-4">
        <span class="text-xs font-semibold">Tables:</span>
        <UTabs class="mt-1" orientation="vertical" variant="pill" :content="false" :items="tables.map((item) => {
                 return {
                   label: item.name,
                   icon: 'i-lucide-table'
                 }
               })" v-model="selectedTableTab" @update:modelValue="(e) => selectTable(e)" :ui=" {
                 trigger: 'items-start text-start w-full'
               }"/>
      </div>

      <!--Main Content-->
      <div class="flex-1">
        <main>
          <div v-if="connection" class=" flex flex-col">
            <div>
              <UTable
                :data="formattedRows"
                :sticky="true"
                :ui=" {
                  root: 'h-[calc(100vh-122px)]',
                  thead: 'sticky top-0 inset-x-0 bg-default z-[2] backdrop-blur-none border-b border-accented',
                  th: 'px-4 py-3.5 text-sm text-highlighted text-left font-semibold border-l border-accented [&:first-child]:border-l-0',
                  td: 'p-4 text-sm text-muted whitespace-nowrap border-l border-accented [&:first-child]:border-l-0',
                  tbody: 'border-b border-accented'
                }"
                class="font-mono border-r border-accented"
              />
            </div>

            <div v-if="rows" class="bg-accent border-t border-accented p-4">
              {{ rows.length }} Total Records
            </div>
          </div>

          <div class="w-full h-[calc(100vh-65px)] flex items-center justify-center" v-else>
            <div class="flex flex-col space-y-2 w-96 border border-accented rounded-xl p-4">
              <span class="mb-8 font-bold">You are not connected</span>

              <UAlert title="Reminder" description="Please ensure you've generated your server bindings and started your SpacetimeDb Server!" variant="soft" color="info"/>

              <UFormField label="Server URI">
                <UInput v-model="serverUri" class="w-full font-mono"/>
              </UFormField>

              <UFormField label="Module Name">
                <UInput v-model="moduleName" class="w-full font-mono"/>
              </UFormField>

              <UButton label="Connect" block @click="connect"/>
            </div>

          </div>
        </main>
      </div>
    </div>
  </div>
</template>