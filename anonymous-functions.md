Un cas pratique des fonctions anonymes avec les GridView widgets de Yii 2.

Il s'agit ici, en lieu et place d'un attribut du modèle d'afficher une autre valeur qui provient d'une méthode.
```php
<?= GridView::widget([
        'dataProvider' => $dataProvider,
        'filterModel' => $searchModel,
        'columns' => [
            ['class' => 'yii\grid\SerialColumn'],

            'reference',
            'libelle',
            'date_debut:date',
            'date_fin:date',
            [
                'attribute' => 'valeur',
                'hAlign' => GridView::ALIGN_RIGHT,
                'value' => function ($model) {
                    return $model->affiche_valeur();
                }
            ],
            [
                'attribute' => 'taxe.libelle',
                'label'=> 'Libellé de la taxe'
            ],
            'formule',

            ['class' => 'yii\grid\ActionColumn'],
        ],
    ]); ?>
  ```
  Ceci signifie normalement que la clé `value` contient la valeur de retour de la fonction anonyme.
  
  La fonction anonyme prend en paramètre la donnée sur laquelle la manipulation se fait. Ici le modèle courant (le dataprovider en contient plusieurs).
  
  Seule la variable `$model` est accessible dans la fonction anonyme. 
  Il peut être nécessaire que la fonction anonyme nécessite une variable se trouvant dans le bloc parent. Dans ce cas, il faut explicitement passer la variable.
  
  ```php
  <?=
            GridView::widget([
                'dataProvider' => $tauxDataProvider,
                'export' => false,
                'headerRowOptions' => ['class' => 'text-center kartik-sheet-style'],
                'showPageSummary' => true,
                'columns' => [
                    ['class' => 'kartik\grid\SerialColumn'],
                    'libelle',
                    [
                        'attribute' => 'valeur',
                        'hAlign' => GridView::ALIGN_RIGHT,
                        'value' => function ($model) {
                            return $model->affiche_valeur();
                        }
                    ],
                    'formule',
                    [
                        'attribute' => 'montant',
                        'format' => 'currency',
                        'pageSummary' => true,
                        'hAlign' => GridView::ALIGN_RIGHT,
                        'value' => function ($data) use ($model){
                            return $model->calcul_montant_droit($data);
                        }
                    ],
                ],
            ]);
            ?>
  ```
  Le mot clé `use` ici sert à passer des arguments provenant du bloc parent.
