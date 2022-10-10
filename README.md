# Лабораторная работа № 2 Интеграция сервиса для получения данных профиля пользователя.
Отчет по лабораторной работе #2 выполнил(а):
- Данькин Сергей Викторович
- РИ-300016

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

## Цель работы
### Cоздание интерактивного приложения и изучение принципов интеграции в него игровых сервисов.

## Задание 1
#### По теме видео практических работ 1-5 повторить реализацию игры на Unity. Привести описание выполненных действий.

## Ход работы

- Видеолекция №1:

Был создан новый [проект Unity]() на версии 2021.3.10f1. Затем в package manager добавил [asset]() драконов (Dragon for Boss Monster : HP) с сайта Unity Asset Story.

![img](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/UnityStory1.jpg)

Скачал и импортировал данный ассет в проект. В нём был выбран красный дракон и добавлен на сцену. Из этого же ассета была взята анимация полёта дракона и повешена на него. После создан [префаб]() яйца дракона - вытянутая сфера. А также [префаб]() щита, с помощью которого будем ловить яйца.

![gif](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/Task1.gif)

- Видеолекция №2:

Выставлено правильное расположение камеры, поменяли проецию на ортографическую и изменили некоторые значения видимости size и far.

![img](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/Camera.jpg)

Следующим этапом стало написание [скрипта EnemyDragon](), с помощью которого дракон сможет двигаться.

```cs
using UnityEngine;

public class EnemyDragon : MonoBehaviour
{
    [SerializeField] private float speed = 1;
    [SerializeField] private float leftRightDistance = 10f;
    [SerializeField] private float chanceDirection = 0.1f;
    
    void Update()
    {
        Vector3 pos = transform.position;
        pos.x += speed * Time.deltaTime;
        transform.position = pos;

        if (pos.x < -leftRightDistance)
        {
            speed = Mathf.Abs(speed);
        }
        else if (pos.x > leftRightDistance)
        {
            speed = -Mathf.Abs(speed);
        }
    }

    private void FixedUpdate()
    {
        if (Random.value < chanceDirection)
        {
            speed *= -1;
        }
    }
}
```
Сам код очень прост. Заводим 3 переменные

- speed - скорость перемещения дракона.
- leftRightDistance - отрезок на котором может летать дракон.
- chanceDirection - шанс, что дракон может поменять направление движения.

В методе Update описано его перемещение и провека, если дракон упирается в левую или правую границу отрезка, то он меняет направление. А в методе FixedUpdate происходит сравнение рандомного числа с нашим шансом. И в случае, если шанс больше, то меняется направление движения.

![gif](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/Task2.gif)

- Видеолекция №3:

В [скрипте EnemyDragon]() написан новый метод DropEgg для выпадения яйца из дракона, который вызывается в методе Start через 2 секунды.

```cs
public class EnemyDragon : MonoBehaviour
{
    [SerializeField] private GameObject dragonEggPrefab;
    [SerializeField] private float timeBetweenEggDrops = 1f;
    
    void Start()
    {
        Invoke("DropEgg", 2f);
    }

    void DropEgg()
    {
        Vector3 myVector = new Vector3(0.0f, 5.0f, 0.0f);
        GameObject egg = Instantiate<GameObject>(dragonEggPrefab);
        egg.transform.position = transform.position + myVector;
        Invoke("DropEgg", timeBetweenEggDrops);
    }
}
```
Сам же префаб яйца был помещен в переменную dragonEggPrefab.

На сцену добавлен игровой объект Plane(Ground) в роли земли. В сам проект добавлены спецэффекты из всё того же Unity Asset Story. Были изменен материал у Ground.

![gif](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/Task3.gif)

- Видеолекция №4:

Был написан [скрипт DragonEgg]() для яйца дракона, чтобы при столкновении с землей (Ground), вызывалась анимация взрыва, а после яйцо становилось невидимым и удалялось на определенной высоте, чтобы снизить нагрузку. К префабу был добавлен компонент Particle System, который отвечает за воспроизведение эффектов.
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DragonEgg : MonoBehaviour
{
    [SerializeField] private static float bottomY = -30;
    private void OnTriggerEnter(Collider other)
    {
        ParticleSystem ps = GetComponent<ParticleSystem>();
        var em = ps.emission;
        em.enabled = true;

        Renderer rend;
        rend = GetComponent<Renderer>();
        rend.enabled = false;    
    }
    void Update()
    {
        if (transform.position.y < bottomY)
        {
            Destroy(this.gameObject);
        }
    }
}
```

Написан [скрипт DragonPicker]() для щита, который повешан на главную камеру. Он отвечает за создание трёх щитов, как жизни, с увеличением размера при старте сцены.

```cs
public class DragonPicker : MonoBehaviour
{
    [SerializeField] private GameObject energyShieldPrefab;
    [SerializeField] private int numEnergyShield = 3;
    [SerializeField] private float energyShieldBottomY = -6f;
    [SerializeField] private float energyShieldRadius = 1.5f;
    void Start()
    {
        for (int i = 1; i <= numEnergyShield; i++)
        {
            GameObject tShieldGo = Instantiate<GameObject>(energyShieldPrefab);
            tShieldGo.transform.position = new Vector3(0, energyShieldBottomY, 0);
            tShieldGo.transform.localScale = new Vector3(1*i, 1*i, 1*i);
        }
    }
}
```
Результат работы:

![gif](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/Task4.gif)

В окне иерархии можно увидеть, что создались 3 щита. А также создаются яйца, которые после выполнения своей роли исчезают.

- Видеолекция №5:

Посмотрели и разобрали консоль яндекс игры. А именно, познакомились с яндекс SDK, посмотрели какие у него есть методы и на каком языке они пишутся. Рассмотрели, как создавать шаблон для будущей игры, в какие поля, что нужно написать.

![img](https://github.com/S1GARETA/UnityLab2/blob/main/Demo%20files/Yandex2.jpg)

## Выводы

Было полезно узнать, откуда брать бесплатные ассеты для проектов, и как с ними работать. Узнал, какие этапы нужно сделать, чтобы загрузить свой проект на яндекс игры.