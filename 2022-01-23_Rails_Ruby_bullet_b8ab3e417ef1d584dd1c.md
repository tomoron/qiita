<!--
title:   【Rails】bulletで検知できないN+1を解消する part1
tags:    Rails,Ruby,bullet
id:      b8ab3e417ef1d584dd1c
private: false
-->
#初めに
Railsのviewの表示速度の低下理由として、partialの使いすぎやN+1問題があります。
このN+1問題を検知するgemとして、有名なのがbulletです
ただし、全てのN+1を検知してくるわけではありせん

###本記事について
bulletで検知できないN+1が発生した時の対処法を僕なりに書いた記事になっております。
間違ったことなどがあれば、ご指摘ください。

#本文
###モデルの定義
まずは、モデルを定義します。
コーヒー豆(bean)とレビュー(review)が1対多で関連付けされています。
beanにはコーヒー豆の名前と国、説明のカラムが存在しています。
reviewにはレビュー内容と香り・甘味・苦味・コク・酸味のカラムが設定されています
香り・甘味・苦味・コク・酸味は1から５の数字で評価していただく形になっています。

```ruby:bean.rb
# Table name: beans
#
#  id            :bigint           not null, primary key
#  name          :string(255)      not null
#  country       :string(255)      not null
#  description   :text(65535)
#  created_at    :datetime         not null
#  updated_at    :datetime         not null

class Bean < ApplicationsRecord
  has_many :reviews

  def evaluation_average
    evaluation = [:acidity,:bitter,:flavor,:rich,:sweet] 
    evaluation.map do |e|
      review.average(e)
    end
    
  end

end

```

`ruby:review.rb

# Table name: reviews
#
#  id                :bigint           not null, primary key
#  content           :text(65535)      not null
#  acidity           :integer
#  bitter            :integer
#  flavor            :integer
#  rich              :integer
#  sweet             :integer
#  created_at        :datetime         not null
#  updated_at        :datetime         not null
#  bean_id        :bigint

class Review < ApplicationRecord
  belongs_to :bean

end
`

viewで香りなどの評価の平均を取ろうとすると、合計でクエリが５つ走ります。

```sql:コンソール
[1] pry(main)> Bean.find(1).evaluation_average
  SELECT AVG(`bean_reviews`.`acidity`) FROM `reviews` WHERE `reviews`.`bean_id` = 1
  SELECT AVG(`bean_reviews`.`bitter`) FROM `reviews` WHERE `reviews`.`bean_id` = 1
  SELECT AVG(`bean_reviews`.`flavor`) FROM `reviews` WHERE `reviews`.`bean_id` = 1
  SELECT AVG(`bean_reviews`.`rich`) FROM `reviews` WHERE `reviews`.`bean_id` = 1
  SELECT AVG(`bean_reviews`.`sweet`) FROM `reviews` WHERE `reviews`.`bean_id` = 1


```

###対処法
selectメソッドのサブクエリを利用することで一つのSQLで対応することができます。
今回はActiveRecord::Relationオブジェクトである必要がないのでpluckメソッドを利用しています。
あと、レビューの関することなのでreviewモデルのメソッドとして定義します。





```ruby:review.rb
class Review < ApplicationRecord
  belongs_to :bean

  def evaluations_average
    pluck("AVG(acidity) AS acidity_sum ,
           AVG(bitter) AS bitter_sum,
           AVG(sweet) AS sweet_sum,
           AVG(rich) AS rich_sum,
           AVG(flavor) AS flavor_sum")[0].map(&:to_s)
  end

end
```

これにより一つのクエリで５つの平均が配列で受け取れます。


#まとめ
そこまでviewの表示速度は変わりませんが、N+1がこれでなくなりました。
これ以上にいい方法があるような気がしますがご指摘があれば、コメントでお願いします。

###参考記事

[Rails: Bulletで検出されないN+1クエリを解消する　- TechRacho](https://techracho.bpsinc.jp/yusiro/2019_12_24/85407)