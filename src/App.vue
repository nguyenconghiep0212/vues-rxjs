<template>
  <div>
    <button class="button">click me</button>
  </div>
</template>

<script  setup>
import { onMounted, ref } from "vue";
import { Observable, throttleTime } from "rxjs";
import { map } from "rxjs/operators";

const users = {
  data: [
    {
      status: "active",
      age: 19,
    },
    {
      status: "active",
      age: 42,
    },
    {
      status: "inactive",
      age: 82,
    },
    {
      status: "active",
      age: 35,
    },
  ],
};

const observable = new Observable((subscriber) => {
  subscriber.next(users);
}).pipe(
  map((value) => {
    console.log("first operator:", value);
    return value.data;
  }),
  map((value) => {
    console.log("second operator:", value);
    return value.filter((e) => e.age < 80);
  }),
  map((value) => {
    console.log("third operator:", value);
    return value.reduce((acc, cur) => acc + cur.age, 0) / value.length;
  })
);

const observer = {
  next: (value) => {
    rando(value);
  },
  error: (er) => {
    console.log("error", er);
  },
  complete: () => {
    console.log("completed");
  },
};
function rando(value) {
  console.log("this is  a custom function", value);
  return value;
}
observable.subscribe(observer);

//
const buttonRef = document.getElementsByClassName("button");
let clickObservable;
// const clickObserver = {
//   next: (value) => {
//     console.log("TODO", value);
//   },
//   error: (er) => {
//     console.log("error", er);
//   },
//   complete: () => {
//     console.log("completed");
//   },
// };
const data = ref([]);
let connection;
onMounted(() => {
  clickObservable = new Observable(function (obs) {
    connection = setInterval(() => {
      obs.next(data.value);
      signalrSim();
    }, 1000);
  })
    .pipe(
      throttleTime(5000),
      map((data) => {
        return data.filter((e) => e.validator > 5);
      })
    )
    .subscribe((value) => {
      console.log("TODO", value);
    });
  // .subscribe(clickObserver);

  buttonRef[0].onclick = function () {
    console.log("unsubscribe");
    clearInterval(connection);
    clickObservable.unsubscribe();
  };
});

function signalrSim() {
  function uuidv4() {
    return "10000000-1000-4000-8000-100000000000".replace(/[018]/g, (c) =>
      (
        +c ^
        (crypto.getRandomValues(new Uint8Array(1))[0] & (15 >> (+c / 4)))
      ).toString(16)
    );
  }
  data.value.push({
    uuid: uuidv4(),
    validator: Math.round(Math.random() * 10),
  });
}
</script> 
