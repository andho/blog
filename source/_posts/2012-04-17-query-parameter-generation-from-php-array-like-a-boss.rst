---
layout: post
title: "Query string generation from PHP array like a boss"
date: 2012-04-17 21:23
comments: true
published: false
categories:
---

if (count($params)) {
	$url .= '?' . implode('&', array_map(function($item) {
		return $item[0] . '=' . $item[1];
	}, array_map(null, array_keys($params), $params)));
}
