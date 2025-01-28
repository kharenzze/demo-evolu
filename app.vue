<script setup lang="ts">
import * as S from "@effect/schema/Schema";
import {
  NonEmptyString1000,
  SqliteBoolean,
  cast,
  database,
  id,
  table,
  // can be also imported from "@evolu/react"
} from "@evolu/common";

// Without React
import { createEvolu } from "@evolu/common-web";

const TodoId = id("Todo");
// It's branded string: string & Brand<"Id"> & Brand<"Todo">
// TodoId type ensures no other ID can be used where TodoId is expected.
type TodoId = typeof TodoId.Type;

const TodoTable = table({
  id: TodoId,
  // Note we can enforce NonEmptyString1000.
  title: NonEmptyString1000,
  // SQLite doesn't support the boolean type, so Evolu uses SqliteBoolean instead.
  isCompleted: S.NullOr(SqliteBoolean),
});

type TodoTable = typeof TodoTable.Type;

const Database = database({
  todo: TodoTable,
});
type Database = typeof Database.Type;

const evolu = createEvolu(Database, {
  initialData: (evolu) => {
    evolu.create("todo", {
      title: S.decodeSync(NonEmptyString1000)("Base Item"),
      isCompleted: false,
    });
  },
});

// Create a typed SQL query. Yes, with autocomplete and type-checking.
const allTodos = evolu.createQuery((db) =>
  db
    .selectFrom("todo")
    .selectAll()
    // SQLite doesn't support the boolean type, but we have `cast` helper.
    .where("isCompleted", "is not", cast(true))
    .orderBy("createdAt")
);

const data = ref<readonly any[]>([]);

const updateData = async () => {
  const { rows } = await evolu.loadQuery(allTodos);
  data.value = rows;
};

const unsubscribeQuery = evolu.subscribeQuery(allTodos)(updateData);

onMounted(() => {
  updateData();
});

onUnmounted(() => {
  unsubscribeQuery();
});

const onClickReset = () => {
  evolu.resetOwner();
};

const onClickCreate = () => {
  const n = data.value.length;
  evolu.create("todo", {
    title: S.decodeSync(NonEmptyString1000)(`New Item ${n}`),
    isCompleted: false,
  });
};
</script>

<template>
  <div class="m-8">
    <div class="flex flex-row gap-4">
      <UButton @click="onClickReset">Reset</UButton>
      <UButton @click="onClickCreate">Create</UButton>
    </div>
    <pre class="mt-8">{{ data }}</pre>
  </div>
</template>
