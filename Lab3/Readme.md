# Lab 3 - Stream processing with Apache Flink
## Тесты
* Тесты для каждого упражнения были пройдены:  
![image](https://user-images.githubusercontent.com/91950488/204253362-28fe511b-b429-4ffb-b4a4-3cb444e25d8f.png)  
![image](https://user-images.githubusercontent.com/91950488/204253428-bb9f46df-e901-4ab8-ba91-0aa274e51c4d.png)  
![image](https://user-images.githubusercontent.com/91950488/204253489-ca4ee819-288d-40a1-abad-47487f0677c9.png)  
![image](https://user-images.githubusercontent.com/91950488/204253578-232b0b9b-cee0-4675-b549-595e895c0b42.png)  

## Решение
### RideCleanisingExercise
#### Задание
The task of the exercise is to filter a data stream of taxi ride records to keep only rides that start and end within New York City. The resulting stream should be printed.
#### Код
```java
private static class NYCFilter implements FilterFunction<TaxiRide> {
    @Override
    public boolean filter(TaxiRide taxiRide) throws Exception {
	    return GeoUtils.isInNYC(taxiRide.startLon, taxiRide.startLat) && GeoUtils.isInNYC(taxiRide.endLon, taxiRide.endLat);
    }
}
```
#### Пояснение
С помощью функции isInNYC, которая определяет принадлежит ли координаты Нью-Йорку, из библиотеки GeoUtils определяем поездки, которые начаты и закончены в Нью-Йорке.

### RidesAndFaresExercise
#### Задание
The goal for this exercise is to enrich TaxiRides with fare information.
#### Код
```java
public static class EnrichmentFunction extends RichCoFlatMapFunction<TaxiRide, TaxiFare, Tuple2<TaxiRide, TaxiFare>> {
    private ValueState<TaxiRide> taxiRideValueState;
    private ValueState<TaxiFare> taxiFareValueState;

    @Override
    public void open(Configuration config) throws Exception {
        ValueStateDescriptor<TaxiRide> taxiRideValueStateDescriptor = new ValueStateDescriptor<TaxiRide>(
                "persistedTaxiRide", TaxiRide.class);
        ValueStateDescriptor<TaxiFare> taxiFareValueStateDescriptor = new ValueStateDescriptor<TaxiFare>(
                "persistedTaxiFare", TaxiFare.class);

        this.taxiRideValueState = getRuntimeContext().getState(taxiRideValueStateDescriptor);
        this.taxiFareValueState = getRuntimeContext().getState(taxiFareValueStateDescriptor);
    }

    @Override
    public void flatMap1(TaxiRide ride, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
        TaxiFare taxiFare = this.taxiFareValueState.value();
        if (taxiFare != null) {
            this.taxiFareValueState.clear();
            out.collect(new Tuple2<>(ride, taxiFare));
        } else {
            this.taxiRideValueState.update(ride);
        }
    }

    @Override
    public void flatMap2(TaxiFare fare, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
        TaxiRide taxiRide = this.taxiRideValueState.value();
        if (taxiRide != null) {
            this.taxiRideValueState.clear();
            out.collect(new Tuple2<>(taxiRide, fare));
        } else {
            this.taxiFareValueState.update(fare);
        }
    }
}
```
#### Пояснение
Реализуя EnrichmentFunction, которая наследуется от RichCoFlatMapFunction, будем соединять пары <TaxiRide, TaxiFare> по ключу rideId. flatMap1 и flatMap2 принимают на вход TaxiRide или TaxiFare соответственно, если в поле класса содержится значение taxiFare или taxiRide соответственно, то применяется out.collect с переданным набором из 2 элементов, иначе поданная на вход сущность записывается в поле класса. Проблема в том, что искать пару будет бесконечно, пока не пройдёт все данные.
### HourlyTipsExerxise
#### Задание
The task of the exercise is to first calculate the total tips collected by each driver, hour by hour, and then from that stream, find the highest tip total in each hour.
#### Код
```java
public class HourlyTipsExercise extends ExerciseBase {

    public static void main(String[] args) throws Exception {

        // read parameters
        ParameterTool params = ParameterTool.fromArgs(args);
        final String input = params.get("input", ExerciseBase.pathToFareData);

        final int maxEventDelay = 60;       // events are out of order by max 60 seconds
        final int servingSpeedFactor = 600; // events of 10 minutes are served in 1 second

        // set up streaming execution environment
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.setParallelism(ExerciseBase.parallelism);

        // start the data generator
        DataStream<TaxiFare> fares = env.addSource(fareSourceOrTest(new TaxiFareSource(input, maxEventDelay, servingSpeedFactor)));

        DataStream<Tuple3<Long, Long, Float>> hourlyTips = fares
                .keyBy((TaxiFare fare) -> fare.driverId)
                .window(TumblingEventTimeWindows.of(Time.hours(1)))
                .process(new AddTips());
        DataStream<Tuple3<Long, Long, Float>> hourlyMax = hourlyTips
                .windowAll(TumblingEventTimeWindows.of(Time.hours(1)))
                .maxBy(2);

        printOrTest(hourlyMax);
        // execute the transformation pipeline
        env.execute("Hourly Tips (java)");
    }

    public static class AddTips extends ProcessWindowFunction<TaxiFare, Tuple3<Long, Long, Float>, Long, TimeWindow> {
        @Override
        public void process(Long key, Context context, Iterable<TaxiFare> fares, Collector<Tuple3<Long, Long, Float>> out) {
            float sumOfTips = 0.0F;
            for (TaxiFare f : fares) {
                sumOfTips += f.tip;
            }
            out.collect(Tuple3.of(context.window().getEnd(), key, sumOfTips));
        }
    }
}
```
#### Пояснение
По ключу fare.driverId с помощью window(TumblingEventTimeWindows.of(Time.hours(1))).process(new AddTips()) получаем чаевые всех водителей за каждый час (время в конце часа в сек, driverId, сумма чаевых). Дальше с проходимся с помощью hourlyTips.timeWindowAll(Time.hours(1)) по каждому часу и выбераем наибольшее значение по 2му полю(чаевым). 
### ExpiringStateExercise
#### Задание
The goal for this exercise is to enrich TaxiRides with fare information.
#### Код
```java
public static class EnrichmentFunction extends KeyedCoProcessFunction<Long, TaxiRide, TaxiFare, Tuple2<TaxiRide, TaxiFare>> {
  private ValueState<TaxiRide> taxiRideValueState;
  private ValueState<TaxiFare> taxiFareValueState;

  @Override
  public void open(Configuration config) throws Exception {
    ValueStateDescriptor<TaxiRide> taxiRideDescriptor = new ValueStateDescriptor<>(
        "persistedTaxiRide", TaxiRide.class);
    ValueStateDescriptor<TaxiFare> taxiFareDescriptor = new ValueStateDescriptor<>(
        "persistedTaxiFare", TaxiFare.class);

    this.taxiRideValueState = getRuntimeContext().getState(taxiRideDescriptor);
    this.taxiFareValueState = getRuntimeContext().getState(taxiFareDescriptor);
  }

  @Override
  public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
    if (this.taxiFareValueState.value() != null) {
      ctx.output(unmatchedFares, this.taxiFareValueState.value());
      this.taxiFareValueState.clear();
    }
    if (this.taxiRideValueState.value() != null) {
      ctx.output(unmatchedRides, this.taxiRideValueState.value());
      this.taxiRideValueState.clear();
    }
  }

  @Override
  public void processElement1(TaxiRide ride, Context context, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
    TaxiFare fare = this.taxiFareValueState.value();
    if (fare != null) {
      this.taxiFareValueState.clear();
      context.timerService().deleteEventTimeTimer(ride.getEventTime());
      out.collect(new Tuple2<>(ride, fare));
    } else {
      this.taxiRideValueState.update(ride);
      context.timerService().registerEventTimeTimer(ride.getEventTime());
    }
  }

  @Override
  public void processElement2(TaxiFare fare, Context context, Collector<Tuple2<TaxiRide, TaxiFare>> out) throws Exception {
    TaxiRide ride = this.taxiRideValueState.value();
    if (ride != null) {
      this.taxiRideValueState.clear();
      context.timerService().deleteEventTimeTimer(fare.getEventTime());
      out.collect(new Tuple2<>(ride, fare));
    } else {
      this.taxiFareValueState.update(fare);
      context.timerService().registerEventTimeTimer(fare.getEventTime());
    }
  }
}
```
#### Пояснение
В этом задании исправляется проблема из 2го задания. Теперь мы ищем пару значений только определённое таймером время, после этого, если не нашли пару, записываем значение в объекты контекста, записанные с тегами unmatchedRides и unmatchedFares. 
