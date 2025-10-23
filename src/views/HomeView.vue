<script setup lang="ts">
import * as moduleBindings from '../spacetime_module_bindings/index';
import {ref, computed, onMounted} from "vue";
import type { NavigationMenuItem } from '@nuxt/ui'

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

const findReducers = (): string[] => {
  // Get all method names from RemoteReducers' prototype
  return Object.getOwnPropertyNames(moduleBindings.RemoteReducers.prototype)
    // Keep only methods that start with 'on' and are followed by a capital letter
    .filter(name => /^on[A-Z]/.test(name))
    // Strip the 'on' prefix to get the reducer name
    .map(name => name.slice(2));
};


const tables = ref<any[]>([])
const reducers = ref<any[]>([])

findTables().forEach((table) => {
  tables.value.push({
    name: table
  })
});

findReducers().forEach((reducer) => {
  reducers.value.push({
    name: reducer
  })
});

const tableSubscription = ref<any | null>();

const safeStringify = (obj: any, space = 2) =>
  JSON.stringify(obj, (_, value) =>
    typeof value === "bigint" ? value.toString() : value,
    space
  );

const selectTable = async(tableName: any) => {
  if (!connection.value) return ;

  selectedReducer.value = null;
  selectedReducerTab.value = 0;
  reducerInfo.value = null;
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

const reducerInfo = ref();

const lowerFirst = str => str.charAt(0).toLowerCase() + str.slice(1);

const selectReducer = async(reducerName: any) => {
  lowerFirst(reducerName)
  if (!connection.value) return ;

  selectedTable.value = null;
  selectedTableTab.value = 0;
  selectedReducer.value = reducerName;

  rows.value = null;

  if (tableSubscription.value) {
    tableSubscription.value = null;
  }

  reducerInfo.value = connection.value.reducers[lowerFirst(reducerName)]
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
const selectedReducer = ref();
const selectedTableTab = ref(0);

const selectedReducerTab = ref(0);

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
  <div class="bg-zinc-100 dark:bg-black p-2">
    <!--Header-->
    <div class="flex justify-between p-4">
      <div class="font-bold flex items-center">Spacetime Panel <UBadge label="v0.3.0" size="xs" variant="soft" class="ml-2"/></div>
      <div class="flex space-x-2">
        <div v-if="connection">
          <UBadge :label="'Connected to ' + serverUri" color="success" variant="soft" class="mr-2 rounded-full"/>
          <UButton label="Disconnect" color="neutral" @click="disconnect"/>
        </div>
        <div v-else>
          <UBadge label="Disconnected" color="error" variant="soft" class="rounded-full"/>
        </div>
        <UColorModeButton/>
        <UButton icon="i-lucide-github" color="neutral" variant="ghost" title="GitHub" to="https://github.com/KeoneSomers/spacetime-panel" target="_blank"/>
      </div>


    </div>

    <div class="flex space-x-2">
      <!--  Sidebar-->
      <UCard variant="soft" :ui=" {body: 'p-2! py-3!'}" class="h-[calc(100vh-80px)] light:bg-white">
        <UNavigationMenu orientation="vertical" variant="pill" :items="[[{label: 'Tables', type: 'label'}, ...tables.map((item) => {
                             return {
                               label: item.name,
                               icon: 'i-lucide-table',
                               active: selectedTable == item.name,
                               onSelect(e: Event) { selectTable(item.name)},
                               disabled: !connection
                             }
                           })], [{label: 'Reducers', type: 'label'}, ...reducers.map((item) => {
                             return {
                               label: item.name,
                               icon: 'i-lucide-square-function',
                               active: selectedReducer == item.name,
                               onSelect(e: Event) { selectReducer(item.name)},
                               disabled: !connection
                             }
                           })]] as NavigationMenuItem[][]" class="w-full"/>
      </UCard>

      <!--Main Content-->
      <UCard variant="soft" class="flex-1 light:bg-white" :ui=" {body: 'p-0!', root: 'mb-0!'}">
        <main>
          <div v-if="connection" class=" flex flex-col">
            <div v-if="reducerInfo" class="p-4">
              <suspense>
                <CodeBlock :content="typeof reducerInfo === 'object'
                           ? JSON.stringify(reducerInfo, null, 2)
                           : typeof reducerInfo === 'function'
                           ? reducerInfo.toString()
                           : String(reducerInfo)"
                />
              </suspense>
            </div>
            <div v-if="selectedTable" class="flex-1">
              <UTable
                :data="formattedRows"
                :sticky="true"
                :ui=" {
                  root: 'h-[calc(100vh-126px)]',
                  thead: 'sticky top-0 inset-x-0 bg-white dark:bg-zinc-900 z-[2] backdrop-blur-none border-b border-accented',
                  th: 'px-4 py-3.5 text-sm text-highlighted text-left font-semibold border-l border-accented [&:first-child]:border-l-0',
                  td: 'p-2 text-sm text-muted whitespace-nowrap border-l border-accented [&:first-child]:border-l-0',
                  tbody: 'border-b border-accented'
                }"
                class="font-mono border-r border-accented"
              />
            </div>

            <div v-if="rows" class="bg-accent border-t border-accented p-3 font-mono text-sm flex items-center bg-accent">
              <div>{{ rows.length }} Total Records</div>
            </div>
          </div>

          <div v-else class="w-full h-[calc(100vh-144px)] flex items-center justify-center">
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
      </UCard>
    </div>
  </div>
</template>