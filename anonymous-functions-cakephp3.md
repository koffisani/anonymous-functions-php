## Ajouter des Champs Calculés

Après vos requêtes, vous aurez peut-être besoin de faire des traitements postérieurs. Si vous voulez ajouter quelques champs calculés ou des données dérivées, vous pouvez utiliser la méthode formatResults(). C’est une façon légère de mapper les ensembles de résultats. Si vous avez besoin de plus de contrôle sur le processus, ou que vous souhaitez réduire les résultats, vous devriez utiliser la fonctionnalité de Map/Reduce à la place. Si vous faîtes une requête d’une liste de personnes, vous pourriez calculer leur âge avec le formateur de résultats:

```php
// En supposant que nous avons construit les champs, les conditions et les contain.
$query->formatResults(function (\Cake\Collection\CollectionInterface  $results) {
    return $results->map(function ($row) {
        $row['age'] = $row['birth_date']->diff(new \DateTime)->y;
        return $row;
    });
});
```

Comme vous pouvez le voir dans l’exemple ci-dessus, les callbacks de formatage récupéreront un ResultSetDecorator en premier argument. Le second argument sera l’instance Query sur laquelle le formateur a été attaché. L’argument $results peut être traversé et modifié autant que nécessaire.

Les formateurs de résultat sont nécessaires pour retourner un objet itérateur, qui sera utilisé comme valeur retournée pour la requête. Les fonctions de formateurs sont appliquées après que toutes les routines Map/Reduce soient exécutées. Les formateurs de résultat peuvent aussi être appliqués dans les associations contain. CakePHP va s’assurer que vos formateurs sont bien scopés. Par exemple, faire ce qui suit fonctionnera comme vous pouvez vous y attendre:

```php
// Dans une méthode dans la table Articles
$query->contain(['Authors' => function ($q) {
    return $q->formatResults(function (\Cake\Collection\CollectionInterface $authors) {
        return $authors->map(function ($author) {
            $author['age'] = $author['birth_date']->diff(new \DateTime)->y;
            return $author;
        });
    });
}]);
```
```php
// Récupère les résultats
$results = $query->all();

// Affiche 29
echo $results->first()->author->age;
```
Comme vu précédemment, les formateurs attachés aux constructeurs de requête associées sont limités pour agir seulement sur les données dans l’association. CakePHP va s’assurer que les valeurs calculées soient insérées dans la bonne entity.

https://book.cakephp.org/3/fr/orm/query-builder.html#ajouter-des-champs-calcules 

## From personnal projects

```php
    /**
     * Obtention des événements récents
     *
     * @return void Rien d'explicite
     */
    public function getEvenementsBetween()
    {
        $debut = $this->request->getQuery("debut");
        $fin = $this->request->getQuery("fin");
        $evenements = (new TableRegistry())
            ->get('Evenements')
            ->find('published')
            ->contain([
                'EvenementImages',
                'Tags',
                'Categories',
            ])
            ->where(function ($exp) use ($debut, $fin) {
                $orConditions = $exp->or([
                    'AND' => [
                        "date_debut <= " => $debut,
                        'date_fin >=' => $fin
                    ]
                ])
                    ->between('date_debut', $debut, $fin)
                    ->between('date_fin', $debut, $fin);

                return $exp->eq('published', true)
                    ->and($orConditions);
            })
            ->orderAsc('date_debut')
            ->orderAsc('date_fin')
            ->map(function ($events) {
                if (isset($events->evenement_images) && !empty($events->evenement_images)) {
                    foreach ($events->evenement_images as $image) {
                        $image->path = Utilities::url($image);
                    }
                } else {
                    $img = new \stdClass();
                    $img->path = Utilities::url(null, 'event');
                    $img->id = "";
                    $img->user_id = null;
                    $img->foreign_key = "";
                    $img->model = "";
                    $img->filename = "";
                    $img->filesize = 0;
                    $img->mime_type = "";
                    $img->extension = "";
                    $img->hash = "";
                    $img->adapter = "";
                    $img->created = "";
                    $img->modified = "";
                    $events->evenement_images[0] = $img;
                }

                    return $events;
                })
                ->toArray();

        $this->set(compact('evenements'));
    }
```
