#eachの入れ子

  items_price = [[“book”,[500, 520, 560]],[[“game”,[5000, 4320, 1330]]]

  items_price.each do |item|  #items_priceを変数itemに代入。
    sum = 0
    item[1].each do |price|  #itemの配列から1つずつ取り出して、自己代入しながらsumを出力。
      sum += price
    end
    puts “#{item[0]}の合計金額#{sum}円”
  end  
