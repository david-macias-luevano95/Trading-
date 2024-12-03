# Traiding Algoritmico

## In this repositorie I will explain strategi for make traiding in python 
using diferent indicator that I make in python whit the support in library

```
#automatizar el proceso 

def support_resistance(df, duration =5, spread =0):
    "El data frame nesesita tener los siguientes nombre de columna: alta, baja, cierre"

    #suport and resistance bilding
    df ['support'] = np.nan
    df['resistence'] = np.nan
    
    #despues de 5descensos consecutivos den minimo anotamos este precio como el soporte
    df.loc[(df['low'].shift(5) > df['low'].shift(4)) &
       (df['low'].shift(4) > df['low'].shift(3)) &
       (df['low'].shift(3) > df['low'].shift(2)) &
       (df['low'].shift(2) > df['low'].shift(1)) &
       (df['low'].shift(1) > df['low'].shift(0)), 'support']= df['low']

#despues de 5 subidas consecutivas del maximo. observamos este precio como la resistencia 

    df.loc[(df['high'].shift(5) < df ['high'].shift(4)) &
        (df['high'].shift(4) < df['high'].shift(3)) &
        (df['high'].shift(3) < df['high'].shift(2)) &
        (df['high'].shift(2) < df['high'].shift(1)) &
        (df['high'].shift(1) < df['high'].shift(0)), 'resistance'] = df ['high'] 

#crear media movil simple de 30 dias 
    df[' SMA fast'] = df['close'].rolling(30).mean()

#crear media movil simple de 60 dias
    df['SMA slow'] = df['close'].rolling(60).mean()


    df [ 'rsi'] = ta.momentum.RSIIndicator(df['close'], window = 10).rsi()

#para hacer la comparacion se compara con el dia anterior 
#RSI yesterday
    df['rsi yesterday'] =  df ['rsi'].shift(1)

    df['signal'] = 0


    df['smooth resistance'] = df['resistance'].fillna(method ='ffill')
    df['smooth support'] = df['support'].fillna(method = 'ffill')

    condition_1_buy = (df['close'].shift(1) < df['smooth resistance'].shift(1)) & \
                    (df['smooth resistance'] *(1+0.5/100) < df['close'])
    
    condition_2_buy = df['SMA fast'] > df['rsi yesterday']

    condition_3_buy = df[ 'rsi'] <df['rsi yesterday']

    condition_1_sell = (df['close'].shift(1) > df['smooth support'].shift(1)) & \
                    (df['smooth support'] *(1+0.5/100) > df['close'].shift(1))

    condition_2_sell = df['SMA fast'] < df['SMA slow']

    condition_3_sell = df['rsi'] > df['rsi yesterday']

    df.loc[condition_1_buy & condition_2_buy & condition_3_buy, 'signal'] =1
    df.loc[condition_1_sell & condition_2_sell & condition_3_sell, 'signal']=-1

#colcular las ganancias
    df['pct'] = df['close'].pct_change(1)

    df['return'] = np.array([df['pct'].shift(i) for i in range(duration)]).sum(axis=0) * (df['signal'].shift(duration))
    df.loc[df['return'] ==-1, 'return'] = df['return']-spread
    df.loc[df['return'] == 1,  'return'] = df['return']-spread

    return df['return']
```
